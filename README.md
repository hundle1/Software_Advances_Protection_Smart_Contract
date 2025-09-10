# Software Advances Protection
----------------------------------------
## Smart-Contract for Non Fungible Software with proof-of-origin
---------------------------------------------------------------------------
## Về architectures
# About Smart Contract:
1. NFS token function(on-chain) :
    - Using ERC extension on each release,binary,build artifact -> each NFS token (for nonfungibke)
    - Metadata for each repo or commit hash, build hash, license type, release timestamp, publischer, owner, craetor.
2. License Smart Contract:
    - Controlling grant/transfer license/revoke (time-limmited/usage/seat-based)
    - Basic function : mintLicense, transferLicense, revokeLicense, renewLicense.
3. Proof-of-origin (POO) registry:
    - Contract hoặc on-chain registry ghi mapping: 
    - Cho phép verify isOrigin(artifactHash) và trace lịch sử chuyển nhượng.
4. Off-chain storage cho artifacts
    - Lưu binary trên IPFS / Arweave; on-chain chỉ lưu hash (tiết kiệm gas).
5. Client SDK / Runtime verification
    - SDK (Node/Go/.NET/Java) tích hợp vào build/runtime để:
        - Kiểm tra ownership (wallet signature / possession of NFS).
        - Fetch license status on-chain.
        - Fail-safe mode nếu không authenticated (configurable).
6. Contributor attribution ledger
    - Mỗi commit/contributor ký GPG/eth-sig; commitHash + authorAddr được ghi vào registry.
    - Hợp đồng có thể phân phối phần trăm royalty tới contributor addresses.
7. Basic web UI / Dashboard
    - Publisher tạo release → mint NFS → upload metadata.
    - Người mua/enterprise xem status license, transfer, renew.
8. Audit trail & transparency
    - Mọi hành động (mint, transfer, revoke, royalty payout) có event logs hiển thị trên dashboard.
9. Governance & license policy
    - Lựa chọn license (MIT/Apache/GPL/dual-license) được gán vào metadata; contract enforce chỉ với thương mại layer.

# Các chức năng nâng cao (Agent / AI / tự động)
A. Vai trò chính của Agent
1. Monitoring & Anomaly Detection
    - Giám sát marketplaces, repos, torrent, IPFS pinning để phát hiện artifact trùng/phi pháp.
    - Dùng ML để phát hiện repackaged builds (binary similarity).
2. License Enforcement
    - Khi phát hiện vi phạm: Agent gửi “notice” on-chain (event) và có thể trigger revokeLicense hoặc freeze nếu hợp đồng cho phép.
    - Thực hiện takedown request tự động (kết hợp hệ thống bên thứ ba).
3. Royalty & Revenue Sharing
    - Khi sale xảy ra, Agent thực hiện payout theo rule (contributors percentages).
4. Dispute Resolution & Evidence Collection
    - Tự thu thập logs, hashes, timestamp, snapshot website để làm bằng chứng on-chain (hash of evidence).
    - Kết nối với arbitration smart contract (DAO/arbitrator).
5. Auto-Updates & Revocation
    - Nếu license hết hạn/lỗi bảo mật nghiêm trọng, Agent có thể đẩy update (signed patch) hoặc switch license state (grace period).
6. Attribution / “Uống nước nhớ nguồn”
    - Khi commercial build được mint, Agent auto-embed provenance metadata (origins + contributor credits).
7. Marketplace Curation
    - Xác thực sellers: Agent đánh dấu marketplace listing là “verified origin” hay “unverified”.
B. Agent architecture notes
    - Agents thường là off-chain services with on-chain hooks (oracles/relayers).
    - Có thể là distributed agents (multi-node) để tránh single point of failure.
    - Agents cần identity: on-chain operator key + reputation system (DAO-governed).
# Kiến trúc & workflow (high level)
1. Dev flow (Open Source → Release)
    - Dev push commit → CI builds artifact → compute artifactHash (binary + metadata) → upload to IPFS → mint NFS token with metadata (commitHash, artifactHash, license) → publish.
2. User flow (Acquire & Run)
    - User purchases/transfers NFS or license via dApp → SDK verifies ownership by wallet signature → runtime unlocks features.
3. Contribution flow (Contribute → Reward)
    - Contributor sign commit (eth address) → contribution recorded on POO registry → on revenue event, royalty pay executed to addresses.
4. Enforcement & Monitoring
    - Agent scans web/IPFS → finds unregistered artifact matching originHash → posts evidence on chain & notifies owner → triggers follow-up (request takedown, escalate to DAO).
5. Dispute flow
    - Subject disputes evidence → arbitration contract used (stake deposit, voting) → final decision triggers on-chain action (payout/revocation).
## Smart contract interfaces & metadata mẫu
# NFS (ERC-721 extension) — key events/functions
```
interface INFS {

  function mintNFS(address to, bytes32 artifactHash, string calldata metadataURI) external returns (uint256);

  function getArtifactHash(uint256 tokenId) external view returns (bytes32);

  function getOrigin(uint256 tokenId) external view returns (address origin);

  event NFSMinted(uint256 indexed tokenId, address indexed origin, bytes32 artifactHash, string metadataURI);

}
```
# LicenseManager
```
interface ILicenseManager {

  function mintLicense(uint256 nfsId, address to, uint256 expiresAt, LicenseType ltype) external returns (uint256 licenseId);

  function revokeLicense(uint256 licenseId) external;

  function isLicenseValid(uint256 licenseId) external view returns (bool);

  event LicenseMinted(uint256 indexed licenseId, uint256 indexed nfsId, address indexed holder, uint256 expiresAt);

  event LicenseRevoked(uint256 indexed licenseId);

}
```
# POO Registry
```
interface IPOORegistry {

  function registerCommit(bytes32 commitHash, address author) external;

  function getAuthors(bytes32 commitHash) external view returns (address[] memory);

  function verifyArtifact(bytes32 artifactHash) external view returns (uint256 originNfsId);

}
```
# Evidence / Dispute (Arbitration)
```
interface IDispute {

  function raiseDispute(bytes32 evidenceHash, uint256 subjectNfsId) external payable;

  function resolveDispute(uint256 disputeId, DisputeOutcome outcome, address[] calldata payees) external;

}
```
# Metadata (JSON stored on IPFS)
```
{

  "name": "MyApp v1.2.3",

  "repo": "https://github.com/org/repo",

  "commitHash": "0xabc123",

  "artifactHash": "0xdef456",

  "license": "MIT",

  "contributors": [

    {"name":"Alice","addr":"0x...","commit":"0xabc"}

  ],

  "buildTimestamp": "2025-09-10T06:00:00Z",

  "originNotes": "open-source core; enterprise features in plugin X"

}
```
# Mô hình kinh tế (tokenomics / rewards)

- Royalty split: % doanh thu cho origin + contributors (on-chain automated).

- Bounty pool / grant: DAO-governed fund trả bounty cho phát hiện vi phạm hoặc hỗ trợ triage bug.

- Reputation token: cho contributor/agent/uploader để khuyến khích đúng hành vi.

# Bảo mật, quyền riêng tư & pháp lý (must-do)

1. Anti-tamper / binary fingerprinting
    - Watermark/binary fingerprint embedding; signed manifests for reproducible builds.
2. Privacy for contributors
    - Contributors có thể dùng pseudonymous addresses; nhưng khi cần thanh toán, KYC có thể yêu cầu cho doanh thu lớn.
3. ZKP cho proprietary parts
    - Dùng ZK to prove certain properties (e.g., license check passed) without revealing secret code.
4. TEE for sensitive runtime
    - Critical verification runs in TEE to avoid tampering by attacker.
5. Legal/licensing compliance
    - Dual-license design; rõ ràng điều khoản commercial vs community; luật sư tech review.

# Threat model & mitigations

    - Replay / counterfeit artifact → store immutable artifactHash; use binary fingerprinting + signed manifest.
    - Fake marketplace → Agent verification badge + on-chain attestation.
    - Key compromise → multisig for mint/revoke; timelocks on critical operations.
    - Collusion / bad-actor agent → distributed agents + DAO governance + slashing.

# MVP checklist (concrete)
    - NFS token contract + deploy testnet.
    - IPFS upload pipeline + signed manifest.
    - LicenseManager contract + basic UI (mint/transfer/revoke).
    - SDK (Node + runtime lib) for auth/verify.
    - POO registry to record commits & contributors.
    - Agent (single node) to watch marketplaces/IPFS and emit evidence events.
    - Royalty payout mechanism.
    - Basic documentation + license templates.

# Tech stack gợi ý
    - Blockchain: Ethereum L2 (Optimism/Arbitrum) hoặc gasless subnet (SKALE) để tiết kiệm phí.
    - Storage: IPFS + Arweave (long term).
    - Runtime: SDK in Rust/Go + electron for desktop app unlock.
    - Agent: Python/Node with ML binary similarity (fuzzy hashing, ssdeep) + web crawlers.
    - ZKP: zkSNARK lib (snarkjs / circom) cho privacy proofs.
    - TEE: Intel SGX or AWS Nitro Enclaves khi cần.

# Example Agent actions (để dễ hình dung)
1. Detect: crawl marketplace → find binary with matching artifactHash → create evidence bundle (screenshot, url, binary hash) → store evidence on IPFS → call raiseDispute(evidenceHash, nfsId) on chain.
2. Enforce: license expired but running → Agent sends update request to runtime or triggers revokeLicense.
3. Reward: when sale happens, Agent triggers distributeRoyalty(nfsId) to contributor addresses.