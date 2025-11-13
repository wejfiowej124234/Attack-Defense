# Eclipse Attack

## 📋 Attack Information
**Category**: P2P Network Attack | **Severity**: 🔴 High (CVSS 7.0)

## 🎯 Attack Principle
Attacker controls all network connections of target node, isolating it from real network and feeding false blockchain data.

## 💀 Attack Flow
```
Normal node:
Node ←→ [Multiple honest nodes] ←→ Blockchain network

Eclipse attack:
Node ←→ [Only attacker nodes] ×××× (isolated from real network)

Result:
- Node only sees blocks provided by attacker
- May be old blocks (double-spend)
- May be fake blocks (fraud)
- Cannot discover real network state
```

## 💀 Attack Consequences

### 1. Double-Spend Assistance
```
1. User node eclipsed
2. Attacker sends transaction to user (confirms on fake chain)
3. User delivers goods/services
4. Attacker ends eclipse
5. User node syncs real chain
6. Discovers transaction never existed
```

### 2. Mining Power Waste
```
Miner node eclipsed
→ Mines on attacker's fake chain
→ Wastes computing power
→ Zero rewards
```

## 🛡️ Defense Measures

### 1. Multiple Node Verification
```typescript
/**
 * Connect to multiple nodes for verification
 */
const TRUSTED_NODES = [
  'https://mainnet.infura.io',
  'https://eth.llamarpc.com',
  'https://cloudflare-eth.com',
  'https://rpc.ankr.com/eth',
];

async function getBlockWithConsensus(blockNumber: number): Promise<Block> {
  // ✅ Get same block from multiple nodes
  const blocks = await Promise.all(
    TRUSTED_NODES.map(url => {
      const provider = new JsonRpcProvider(url);
      return provider.getBlock(blockNumber);
    })
  );
  
  // Verify consistency
  const hashes = blocks.map(b => b.hash);
  const uniqueHashes = new Set(hashes);
  
  if (uniqueHashes.size > 1) {
    throw new Error('Node data inconsistent! Possible eclipse attack!');
  }
  
  return blocks[0];
}
```

### 2. Peer Diversity
```typescript
/**
 * Ensure connection to diverse nodes
 */
class NodeManager {
  private nodes: string[] = [];
  private readonly MIN_NODES = 8;
  
  async ensureDiversity(): Promise<void> {
    // ✅ Connect to nodes from different ISPs, regions
    const diverseNodes = await discoverNodes({
      minDifferentISPs: 3,
      minDifferentCountries: 2,
      minDifferentDatacenters: 3,
    });
    
    this.nodes = diverseNodes.slice(0, this.MIN_NODES);
  }
  
  async broadcastTransaction(tx: Transaction): Promise<void> {
    // ✅ Broadcast to all nodes
    await Promise.all(
      this.nodes.map(node => sendToNode(node, tx))
    );
  }
}
```

### 3. Block Hash Chain Verification
```typescript
/**
 * Verify block hash chain
 */
async function verifyBlockChain(from: number, to: number): Promise<boolean> {
  for (let i = from; i < to; i++) {
    const block = await provider.getBlock(i);
    const nextBlock = await provider.getBlock(i + 1);
    
    // ✅ Verify hash chain
    if (nextBlock.parentHash !== block.hash) {
      console.error(`Blockchain break at ${i}`);
      return false;
    }
  }
  
  return true;
}
```

### 4. Network Health Check
```typescript
/**
 * Detect eclipse attack signs
 */
async function detectEclipse(): Promise<boolean> {
  // 1. Check peer count
  const peerCount = await web3.eth.net.getPeerCount();
  if (peerCount < 3) {
    console.warn('Too few peers, possible eclipse');
    return true;
  }
  
  // 2. Check latest block time
  const latestBlock = await provider.getBlock('latest');
  const now = Date.now() / 1000;
  const blockAge = now - latestBlock.timestamp;
  
  if (blockAge > 300) { // 5 minutes
    console.warn('Block too old, possibly isolated');
    return true;
  }
  
  // 3. Compare with block explorers
  const ourLatest = latestBlock.number;
  const etherscanLatest = await etherscan.getLatestBlock();
  
  if (Math.abs(ourLatest - etherscanLatest) > 10) {
    console.warn('Large block height difference, possible eclipse');
    return true;
  }
  
  return false;
}
```

### 5. Use Trusted RPC Providers
```typescript
// ✅ Use mainstream RPC services
const TRUSTED_RPC_PROVIDERS = [
  'https://mainnet.infura.io/v3/YOUR_KEY',
  'https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY',
  'https://cloudflare-eth.com',
];

// These providers have multiple nodes, hard to eclipse
```

## ✅ Light Node Special Attention
```typescript
// Light nodes only download block headers, easier to eclipse

// Defense:
// 1. Connect to multiple full nodes
// 2. Compare data from multiple sources
// 3. Use checkpoints
```

## ✅ Best Practices
```
□ Connect to multiple diverse nodes
□ Use trusted RPC providers
□ Verify block hash chains
□ Monitor peer count
□ Check block freshness
□ Compare with block explorers
```

**Created**: November 11, 2025

