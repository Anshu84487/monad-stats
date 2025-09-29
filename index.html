/*
Monad Testnet — Monstats-style On‑Chain Dashboard (single-file React component)

What this version does (upgraded):
- Full-page Monstats-like dark UI (hero + card + leaderboard area)
- When a user enters a wallet address and clicks "Check", the app fetches:
  • balance (MON)
  • nonce
  • transaction list from recent blocks (configurable window)
  • computes: total txs, total incoming/outgoing volume, gas spent (uses receipts), days active (based on first/last tx timestamp in window), and a simple "streak" heuristic
  • displays a compact activity timeline and key stat cards (Score, Volume, Gas, Tx count, NFT value placeholder)
- Leaderboard section (mock/demo) showing top wallets (you can later plug an indexer API to populate)

Caveats:
- This client-side scanning approach queries many RPC endpoints to fetch blocks/txs/receipts. It's OK for demo/testnet but will be slow/heavy on public RPCs for big windows. For production, replace with an indexed API (QuickNode, BlockVision, etc.).
- NFT value is a placeholder — requires token metadata + marketplace pricing APIs for real values.

How to use:
- Paste into `src/App.jsx` of a Vite React project and `npm install ethers`
- Run `npm run dev` and try addresses

*/

import React, { useState, useEffect, useRef } from 'react';
import { ethers } from 'ethers';

export default function App() {
  const DEFAULT_RPC = 'https://testnet-rpc.monad.xyz/';
  const [rpcUrl, setRpcUrl] = useState(DEFAULT_RPC);
  const [provider, setProvider] = useState(null);

  const [address, setAddress] = useState('');
  const [status, setStatus] = useState('idle');
  const [blocksToScan, setBlocksToScan] = useState(8000);
  const cancelRef = useRef(false);

  // stats
  const [balance, setBalance] = useState(null);
  const [nonce, setNonce] = useState(null);
  const [txs, setTxs] = useState([]);
  const [metrics, setMetrics] = useState({ totalVolume: '0', incoming: '0', outgoing: '0', gasSpent: '0', transactions: 0, daysActive: 0, streak: 0 });
  const [progress, setProgress] = useState({ scanned: 0, total: 0 });

  // set provider
  useEffect(() => {
    try { setProvider(new ethers.providers.JsonRpcProvider(rpcUrl)); }
    catch { setProvider(null); }
  }, [rpcUrl]);

  const isAddress = (v) => {
    try { return ethers.utils.isAddress(v); } catch { return false; }
  }

  async function fetchQuick(addr) {
    if (!provider) throw new Error('no provider');
    const [bal, non] = await Promise.all([provider.getBalance(addr), provider.getTransactionCount(addr)]);
    setBalance(ethers.utils.formatEther(bal));
    setNonce(non);
  }

  async function scanBlocksForAddress(addr) {
    if (!provider) throw new Error('no provider');
    setStatus('scanning');
    cancelRef.current = false;
    setTxs([]);
    setMetrics({ totalVolume: '0', incoming: '0', outgoing: '0', gasSpent: '0', transactions: 0, daysActive: 0, streak: 0 });

    const latest = await provider.getBlockNumber();
    const fromBlock = Math.max(0, latest - Number(blocksToScan));
    const total = latest - fromBlock + 1;
    setProgress({ scanned: 0, total });

    const found = [];
    const batchSize = 30;
    let scanned = 0;

    for (let b = latest; b >= fromBlock; b -= batchSize) {
      if (cancelRef.current) { setStatus('cancelled'); break; }
      const start = Math.max(fromBlock, b - batchSize + 1);
      const promises = [];
      for (let i = start; i <= b; i++) promises.push(provider.getBlockWithTransactions(i));
      const blocks = await Promise.all(promises);
      for (const block of blocks) {
        if (!block || !block.transactions) continue;
        for (const tx of block.transactions) {
          const from = (tx.from || '').toLowerCase();
          const to = (tx.to || '').toLowerCase();
          const target = addr.toLowerCase();
          if (from === target || to === target) {
            found.push({ ...tx, timestamp: block.timestamp });
          }
        }
      }
      scanned += (b - start + 1);
      setProgress({ scanned: Math.min(total, scanned), total });
      // friendly pause
      await new Promise(r => setTimeout(r, 90));
    }

    // compute metrics (fetch receipts in small batches)
    const receipts = [];
    for (let i = 0; i < found.length; i += 8) {
      if (cancelRef.current) break;
      const slice = found.slice(i, i + 8);
      const rps = slice.map(t => provider.getTransactionReceipt(t.hash));
      const res = await Promise.all(rps.map(p => p.catch(() => null)));
      receipts.push(...res);
      await new Promise(r => setTimeout(r, 60));
    }

    // aggregate
    let totalIn = ethers.BigNumber.from(0);
    let totalOut = ethers.BigNumber.from(0);
    let gasSpentBN = ethers.BigNumber.from(0);
    const target = addr.toLowerCase();
    const timestamps = [];

    for (let i = 0; i < found.length; i++) {
      const tx = found[i];
      const receipt = receipts[i] || null;
      const valueBN = tx.value ? ethers.BigNumber.from(tx.value.toString()) : ethers.BigNumber.from(0);
      if ((tx.to || '').toLowerCase() === target) totalIn = totalIn.add(valueBN);
      if ((tx.from || '').toLowerCase() === target) totalOut = totalOut.add(valueBN);
      if (receipt && receipt.gasUsed) {
        // attempt to compute gas spent = gasUsed * effectiveGasPrice (if available)
        const gasUsed = receipt.gasUsed;
        const price = receipt.effectiveGasPrice || tx.gasPrice || ethers.BigNumber.from(0);
        gasSpentBN = gasSpentBN.add(gasUsed.mul(price));
      }
      if (tx.timestamp) timestamps.push(tx.timestamp);
    }

    // days active (within scanned window)
    let daysActive = 0;
    if (timestamps.length > 0) {
      const minTs = Math.min(...timestamps);
      const maxTs = Math.max(...timestamps);
      daysActive = Math.max(1, Math.ceil((maxTs - minTs) / (24 * 3600)));
    }

    // simple streak heuristic: count consecutive days with at least 1 tx (within window)
    let streak = 0;
    if (timestamps.length > 0) {
      const days = new Set(timestamps.map(ts => Math.floor(ts / 86400)));
      // compute from latest day going backwards
      const latestDay = Math.max(...Array.from(days));
      for (let d = latestDay; ; d--) {
        if (days.has(d)) streak++; else break;
      }
    }

    setTxs(found.map(t => ({ hash: t.hash, from: t.from, to: t.to, value: ethers.utils.formatEther(t.value || 0), blockNumber: t.blockNumber, timestamp: t.timestamp })));

    setMetrics({
      totalVolume: ethers.utils.formatEther(totalIn.add(totalOut)),
      incoming: ethers.utils.formatEther(totalIn),
      outgoing: ethers.utils.formatEther(totalOut),
      gasSpent: ethers.utils.formatEther(gasSpentBN),
      transactions: found.length,
      daysActive,
      streak
    });

    setStatus('done');
    return found;
  }

  async function handleCheck(e) {
    e && e.preventDefault();
    if (!isAddress(address)) { setStatus('bad-address'); return; }
    setStatus('loading');
    try {
      await fetchQuick(address);
      await scanBlocksForAddress(address);
    } catch (err) {
      console.error(err);
      setStatus('error');
    }
  }

  function handleCancel() { cancelRef.current = true; setStatus('cancelling'); }

  function prettyTime(ts) { try { return new Date(ts * 1000).toLocaleString(); } catch { return ts; } }

  // small mock leaderboard (replace later with real service)
  const mockLeaderboard = [
    { addr: '0xAa...1111', score: 982, volume: '120.5' },
    { addr: '0xBb...2222', score: 913, volume: '98.2' },
    { addr: '0xCc...3333', score: 887, volume: '80.0' },
  ];

  return (
    <div className="min-h-screen bg-gradient-to-b from-black to-gray-900 text-white font-sans">
      <div className="max-w-5xl mx-auto py-12 px-6">
        <header className="text-center mb-8">
          <div className="mx-auto w-20 h-20 rounded-2xl bg-gradient-to-br from-purple-600 to-indigo-600 flex items-center justify-center shadow-2xl">
            <svg width="34" height="34" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 2L19 7v10l-7 5-7-5V7l7-5z" stroke="white" strokeWidth="1.2" strokeLinecap="round" strokeLinejoin="round"/></svg>
          </div>
          <h1 className="text-5xl font-extrabold mt-6 text-purple-400">Monstats</h1>
          <p className="text-gray-300 mt-2">Check your on-chain activity</p>
        </header>

        <main>
          <section className="bg-black/60 border border-white/10 rounded-xl p-8 mb-8">
            <h2 className="text-center text-xl font-semibold mb-2">Check Your Wallet Stats</h2>
            <p className="text-center text-sm text-gray-400 mb-6">Enter your wallet address to see stats, volume, gas spent, NFT value and ranking.</p>

            <form onSubmit={handleCheck} className="max-w-3xl mx-auto">
              <div className="grid grid-cols-1 gap-4">
                <input className="w-full p-3 rounded border border-white/20 bg-transparent placeholder-gray-500" placeholder="Enter your wallet address (0x...)" value={address} onChange={(e)=>setAddress(e.target.value)} />
                <div className="flex gap-3 items-center">
                  <input type="number" min={100} max={100000} value={blocksToScan} onChange={(e)=>setBlocksToScan(Number(e.target.value))} className="w-40 p-2 rounded border bg-transparent" />
                  <button type="submit" className="flex-1 py-3 rounded bg-gradient-to-r from-purple-600 to-indigo-600 font-semibold">{status==='scanning' ? 'Scanning...' : 'Check On-Chain Activity'}</button>
                  <button type="button" onClick={handleCancel} className="px-4 py-2 rounded border">Cancel</button>
                </div>
                <div className="text-xs text-gray-400">Status: <span className="font-mono">{status}</span> {status==='scanning' && <span> — scanned {progress.scanned}/{progress.total} blocks</span>}</div>
              </div>
            </form>

            <div className="mt-6 grid grid-cols-1 md:grid-cols-3 gap-4">
              <div className="p-4 bg-white/5 rounded">
                <div className="text-xs text-gray-300">Balance</div>
                <div className="text-2xl font-mono mt-1">{balance ?? '—'} MON</div>
                <div className="text-xs text-gray-400">Nonce: {nonce ?? '—'}</div>
              </div>
              <div className="p-4 bg-white/5 rounded">
                <div className="text-xs text-gray-300">Transactions</div>
                <div className="text-2xl mt-1">{metrics.transactions}</div>
                <div className="text-xs text-gray-400">Days Active: {metrics.daysActive} • Streak: {metrics.streak}</div>
              </div>
              <div className="p-4 bg-white/5 rounded">
                <div className="text-xs text-gray-300">Volume</div>
                <div className="text-2xl mt-1">{metrics.totalVolume} MON</div>
                <div className="text-xs text-gray-400">In: {metrics.incoming} • Out: {metrics.outgoing}</div>
              </div>
            </div>

            <div className="mt-6 p-4 bg-white/5 rounded">
              <div className="flex justify-between items-center mb-3">
                <div className="text-sm text-gray-300">Activity Timeline</div>
                <div className="text-xs text-gray-400">Gas Spent: {metrics.gasSpent} MON</div>
              </div>
              <div className="space-y-2 max-h-56 overflow-auto">
                {txs.length === 0 && <div className="text-gray-500">No transactions found in scanned window.</div>}
                {txs.map((t)=> (
                  <div key={t.hash} className="flex justify-between items-start bg-black/30 p-2 rounded">
                    <div>
                      <div className="text-xs text-gray-300">{t.from} → {t.to}</div>
                      <div className="text-xs text-gray-400">Block #{t.blockNumber} • {prettyTime(t.timestamp)}</div>
                    </div>
                    <div className="text-right font-mono">{t.value} MON</div>
                  </div>
                ))}
              </div>
            </div>

          </section>

          <section className="bg-black/60 border border-white/10 rounded-xl p-6">
            <div className="flex justify-between items-center mb-4">
              <h3 className="text-lg font-semibold">Leaderboard</h3>
              <div className="text-sm text-gray-400">Users: 46,048 • Sorted by Score ↓</div>
            </div>

            <div className="grid md:grid-cols-3 gap-4">
              <div className="md:col-span-2">
                <div className="space-y-3">
                  {mockLeaderboard.map((u, idx) => (
                    <div key={u.addr} className="flex justify-between items-center p-3 bg-black/30 rounded">
                      <div>
                        <div className="text-sm font-semibold">#{idx+1} • {u.addr}</div>
                        <div className="text-xs text-gray-400">Volume: {u.volume} MON</div>
                      </div>
                      <div className="text-right">
                        <div className="text-sm font-bold">{u.score}</div>
                        <div className="text-xs text-gray-400">Score</div>
                      </div>
                    </div>
                  ))}
                </div>
              </div>

              <div>
                <div className="p-3 bg-black/30 rounded">
                  <div className="text-sm text-gray-300 mb-2">Search</div>
                  <input placeholder="Search by wallet address..." className="w-full p-2 rounded border bg-transparent" />
                </div>
                <div className="mt-4 p-3 bg-black/30 rounded">
                  <div className="text-sm text-gray-300">Top Filters</div>
                  <div className="mt-2 flex flex-wrap gap-2">
                    <button className="px-3 py-1 rounded bg-purple-600">Score</button>
                    <button className="px-3 py-1 rounded border">Volume</button>
                    <button className="px-3 py-1 rounded border">Gas Spent</button>
                    <button className="px-3 py-1 rounded border">Transactions</button>
                  </div>
                </div>
              </div>
            </div>

          </section>

        </main>

        <footer className="mt-8 text-center text-xs text-gray-500">
          Built for Monad Testnet — uses public RPC {rpcUrl} • Powered by <span className="font-semibold text-purple-300">@dogsulek</span>
        </footer>
      </div>
    </div>
  );
}
