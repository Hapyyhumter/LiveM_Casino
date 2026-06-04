# LiveM_Casino
Casino LiveM

```jsx
import { useState, useEffect, useRef, useCallback, useMemo } from "react";

const HOUSE_EDGE = 0.025;
const SUITS = ["♠","♥","♦","♣"];
const RANKS = ["A","2","3","4","5","6","7","8","9","10","J","Q","K"];

function makeDeck() {
  const d = [];
  for (const s of SUITS) for (const r of RANKS) d.push({ s, r });
  return d.sort(() => Math.random() - 0.5);
}

function cardVal(r) {
  if (["J","Q","K"].includes(r)) return 10;
  if (r === "A") return 11;
  return parseInt(r);
}

function handVal(hand) {
  let v = hand.reduce((s, c) => s + cardVal(c.r), 0);
  let aces = hand.filter(c => c.r === "A").length;
  while (v > 21 && aces > 0) { v -= 10; aces--; }
  return v;
}

const ROULETTE_COLORS = {
  0:"green",1:"red",2:"black",3:"red",4:"black",5:"red",6:"black",7:"red",8:"black",9:"red",
  10:"black",11:"black",12:"red",13:"black",14:"red",15:"black",16:"red",17:"black",18:"red",
  19:"red",20:"black",21:"red",22:"black",23:"red",24:"black",25:"red",26:"black",27:"red",
  28:"black",29:"black",30:"red",31:"black",32:"red",33:"black",34:"red",35:"black",36:"red"
};

const NAMES = ["Steve","Alex","Notch","HeroB","xX_Dark","CreeperK","DiamondP","EnderG","NightW","BlazeFX"];
const GAMES_LIST = ["Blackjack","Coinflip","Roulette","Crash"];

function rndName() { return NAMES[Math.floor(Math.random()*NAMES.length)]; }
function rndGame() { return GAMES_LIST[Math.floor(Math.random()*GAMES_LIST.length)]; }
function rndAmt() { return Math.floor(Math.random()*2000+50); }
function genCode() { return Math.random().toString(36).substr(2,8).toUpperCase(); }

const css = `
  @import url('https://fonts.googleapis.com/css2?family=Rajdhani:wght@400;500;700&family=Share+Tech+Mono&display=swap');
  *{box-sizing:border-box;margin:0;padding:0}
  :root{
    --bg:#0a0e14;--bg2:#111722;--bg3:#161d2a;--bg4:#1c2535;
    --green:#00ff9c;--gold:#ffaa00;--red:#ff4466;--blue:#4488ff;--purple:#a855f7;
    --text:#e8edf5;--muted:#6b7a8f;--border:#1e2d42;
    --card-red:#e74c6f;--card-black:#e8edf5;
  }
  body{background:var(--bg);color:var(--text);font-family:'Rajdhani',sans-serif;min-height:100vh}
  .app{display:flex;flex-direction:column;min-height:100vh}
  .nav{background:var(--bg2);border-bottom:1px solid var(--border);padding:0 24px;display:flex;align-items:center;gap:0;position:sticky;top:0;z-index:100}
  .logo{font-size:22px;font-weight:700;color:var(--green);letter-spacing:2px;margin-right:32px;padding:16px 0}
  .nav-btn{background:none;border:none;color:var(--muted);font-family:'Rajdhani',sans-serif;font-size:15px;font-weight:500;padding:18px 16px;cursor:pointer;border-bottom:2px solid transparent;transition:all .15s;letter-spacing:.5px}
  .nav-btn:hover{color:var(--text)}
  .nav-btn.active{color:var(--green);border-bottom-color:var(--green)}
  .balance-pill{margin-left:auto;background:var(--bg3);border:1px solid var(--border);padding:6px 16px;border-radius:20px;font-size:14px;color:var(--gold);font-family:'Share Tech Mono',monospace;letter-spacing:1px}
  .main{flex:1;display:flex;gap:0}
  .content{flex:1;padding:28px;max-width:820px}
  .sidebar{width:240px;background:var(--bg2);border-left:1px solid var(--border);padding:16px;flex-shrink:0;overflow:hidden}
  .section-title{font-size:11px;letter-spacing:2px;color:var(--muted);text-transform:uppercase;margin-bottom:12px}
  .game-card{background:var(--bg2);border:1px solid var(--border);border-radius:12px;padding:24px;margin-bottom:20px}
  .game-title{font-size:20px;font-weight:700;letter-spacing:1px;margin-bottom:4px;color:var(--text)}
  .game-sub{font-size:13px;color:var(--muted);margin-bottom:20px}
  .bet-row{display:flex;align-items:center;gap:10px;margin-bottom:16px;flex-wrap:wrap}
  .bet-label{font-size:13px;color:var(--muted);min-width:40px}
  .bet-input{background:var(--bg3);border:1px solid var(--border);color:var(--text);padding:8px 12px;border-radius:8px;font-family:'Share Tech Mono',monospace;font-size:14px;width:110px}
  .bet-input:focus{outline:none;border-color:var(--green)}
  .chip{background:var(--bg3);border:1px solid var(--border);color:var(--text);padding:6px 12px;border-radius:6px;font-size:13px;cursor:pointer;transition:all .1s}
  .chip:hover{border-color:var(--green);color:var(--green)}
  .btn{padding:10px 22px;border-radius:8px;font-family:'Rajdhani',sans-serif;font-size:15px;font-weight:700;letter-spacing:1px;cursor:pointer;border:none;transition:all .15s}
  .btn-green{background:var(--green);color:#0a0e14}
  .btn-green:hover{background:#00cc7a}
  .btn-green:disabled{background:#1a3d2e;color:#2d6b4f;cursor:not-allowed}
  .btn-outline{background:transparent;border:1px solid var(--border);color:var(--text)}
  .btn-outline:hover{border-color:var(--green);color:var(--green)}
  .btn-red{background:var(--red);color:#fff}
  .btn-red:hover{background:#cc2244}
  .btn-gold{background:var(--gold);color:#0a0e14}
  .btn-gold:hover{background:#cc8800}
  .btn-purple{background:var(--purple);color:#fff}
  .btn-purple:hover{background:#9333ea}
  .btn-purple:disabled{background:#2d1a4a;color:#6b3a8f;cursor:not-allowed}
  .result-box{padding:14px 18px;border-radius:10px;font-size:16px;font-weight:700;letter-spacing:.5px;margin-top:12px;text-align:center}
  .result-win{background:#0d3320;border:1px solid var(--green);color:var(--green)}
  .result-lose{background:#2d0f18;border:1px solid var(--red);color:var(--red)}
  .result-push{background:#2a2400;border:1px solid var(--gold);color:var(--gold)}
  .result-info{background:#0d1a33;border:1px solid var(--blue);color:var(--blue)}
  .cards-row{display:flex;gap:8px;flex-wrap:wrap;margin:12px 0}
  .card{width:52px;height:74px;border-radius:8px;border:1px solid #2a3a50;background:var(--bg4);display:flex;flex-direction:column;align-items:center;justify-content:center;font-size:18px;font-weight:700;font-family:'Share Tech Mono',monospace;transition:transform .15s}
  .card-r{color:var(--card-red)}
  .card-b{color:var(--card-black)}
  .card-back{color:var(--border);font-size:22px}
  .hand-label{font-size:12px;color:var(--muted);margin-bottom:6px;letter-spacing:1px}
  .btn-row{display:flex;gap:8px;flex-wrap:wrap;margin-top:12px}
  .coin-area{display:flex;flex-direction:column;align-items:center;gap:16px;padding:16px 0}
  .coin{width:90px;height:90px;border-radius:50%;border:3px solid var(--border);display:flex;align-items:center;justify-content:center;font-size:32px;font-weight:700;font-family:'Share Tech Mono',monospace;transition:all .3s}
  .coin-h{background:#1a2e10;border-color:var(--green);color:var(--green)}
  .coin-t{background:#1e1800;border-color:var(--gold);color:var(--gold)}
  .coin-idle{background:var(--bg3);color:var(--muted)}
  .coin-spinning{animation:spin .6s linear infinite}
  .side-btns{display:flex;gap:10px;justify-content:center}
  .side-btn{flex:1;padding:12px;border-radius:8px;font-family:'Rajdhani',sans-serif;font-size:15px;font-weight:700;cursor:pointer;border:2px solid;transition:all .15s;background:transparent}
  .side-btn-h{border-color:var(--green);color:var(--green)}
  .side-btn-h.chosen,.side-btn-h:hover{background:var(--green);color:#0a0e14}
  .side-btn-t{border-color:var(--gold);color:var(--gold)}
  .side-btn-t.chosen,.side-btn-t:hover{background:var(--gold);color:#0a0e14}
  .roulette-wheel{width:140px;height:140px;border-radius:50%;border:4px solid var(--border);display:flex;align-items:center;justify-content:center;font-size:36px;font-weight:700;font-family:'Share Tech Mono',monospace;margin:0 auto 16px;transition:all .4s}
  .rw-idle{background:var(--bg3);color:var(--muted);border-color:var(--border)}
  .rw-red{background:#3d0f1a;border-color:var(--red);color:var(--red)}
  .rw-black{background:var(--bg4);border-color:#666;color:#ccc}
  .rw-green{background:#0d2a0f;border-color:var(--green);color:var(--green)}
  .roulette-bets{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-bottom:14px}
  .rbet{padding:10px;border-radius:8px;border:1px solid var(--border);background:var(--bg3);cursor:pointer;text-align:center;font-size:13px;font-weight:700;letter-spacing:.5px;transition:all .15s;color:var(--text)}
  .rbet:hover{border-color:var(--green)}
  .rbet.sel{border-width:2px}
  .rbet-red.sel{border-color:var(--red);color:var(--red);background:#2d0f18}
  .rbet-black.sel{border-color:#aaa;color:#ddd;background:#1a1a1a}
  .rbet-green.sel{border-color:var(--green);color:var(--green);background:#0d3320}
  .rbet-even.sel,.rbet-odd.sel,.rbet-low.sel,.rbet-high.sel{border-color:var(--blue);color:var(--blue);background:#0d1a33}
  .feed-item{padding:10px 0;border-bottom:1px solid var(--border);font-size:13px;display:flex;justify-content:space-between;align-items:center}
  .feed-name{color:var(--text);font-weight:500}
  .feed-game{color:var(--muted);font-size:12px}
  .feed-win{color:var(--green);font-family:'Share Tech Mono',monospace;font-size:13px}
  .feed-lose{color:var(--red);font-family:'Share Tech Mono',monospace;font-size:13px}
  .wallet-grid{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-bottom:20px}
  .wcard{background:var(--bg3);border:1px solid var(--border);border-radius:10px;padding:16px}
  .wcard-label{font-size:12px;color:var(--muted);letter-spacing:1px;text-transform:uppercase;margin-bottom:6px}
  .wcard-val{font-size:26px;font-weight:700;font-family:'Share Tech Mono',monospace;color:var(--gold)}
  .tx-row{padding:10px 0;border-bottom:1px solid var(--border);display:flex;justify-content:space-between;font-size:13px;gap:8px}
  .tx-plus{color:var(--green);font-family:'Share Tech Mono',monospace}
  .tx-minus{color:var(--red);font-family:'Share Tech Mono',monospace}
  .spinning{animation:spin .6s linear infinite}
  @keyframes spin{from{transform:rotate(0deg)}to{transform:rotate(360deg)}}
  .fade-in{animation:fadeIn .3s ease}
  @keyframes fadeIn{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}
  .hero-title{font-size:52px;font-weight:700;letter-spacing:4px;color:var(--green);line-height:1}
  .hero-sub{font-size:18px;color:var(--muted);margin-top:8px;letter-spacing:1px}
  .games-grid{display:grid;grid-template-columns:1fr 1fr;gap:16px;margin-top:28px}
  .game-thumb{background:var(--bg2);border:1px solid var(--border);border-radius:12px;padding:24px;cursor:pointer;transition:all .2s;text-align:center}
  .game-thumb:hover{border-color:var(--green);transform:translateY(-2px)}
  .stats-row{display:flex;gap:12px;margin-top:20px}
  .stat{background:var(--bg3);border:1px solid var(--border);border-radius:8px;padding:12px 16px;flex:1;text-align:center}
  .stat-val{font-size:22px;font-weight:700;font-family:'Share Tech Mono',monospace;color:var(--green)}
  .stat-label{font-size:11px;color:var(--muted);letter-spacing:1px;text-transform:uppercase;margin-top:2px}
  .discord-box{background:#1a1f35;border:1px solid #5865f2;border-radius:12px;padding:20px;margin-bottom:16px}
  .discord-title{font-size:16px;font-weight:700;color:#5865f2;letter-spacing:1px;margin-bottom:4px;display:flex;align-items:center;gap:8px}
  .discord-sub{font-size:13px;color:var(--muted);margin-bottom:16px}
  .code-block{background:#0d1117;border:1px solid var(--border);border-radius:8px;padding:12px 16px;font-family:'Share Tech Mono',monospace;font-size:14px;color:var(--green);letter-spacing:1px;display:flex;justify-content:space-between;align-items:center;margin-bottom:10px}
  .copy-btn{background:var(--bg3);border:1px solid var(--border);color:var(--muted);padding:4px 12px;border-radius:6px;font-size:12px;cursor:pointer;font-family:'Rajdhani',sans-serif}
  .copy-btn:hover{border-color:var(--green);color:var(--green)}
  .step-list{list-style:none;display:flex;flex-direction:column;gap:10px}
  .step-item{display:flex;gap:12px;align-items:flex-start;font-size:13px;color:var(--text)}
  .step-num{background:var(--bg4);border:1px solid var(--border);width:22px;height:22px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:11px;font-weight:700;flex-shrink:0;color:var(--green)}
  .dep-pending{background:#1a2010;border:1px solid var(--green);border-radius:10px;padding:16px;margin-top:12px}
  .dep-pending-title{font-size:13px;font-weight:700;color:var(--green);margin-bottom:8px}
  .dep-pending-row{display:flex;justify-content:space-between;font-size:13px;padding:4px 0;border-bottom:1px solid var(--border)}
  .verify-input{background:var(--bg3);border:1px solid var(--border);color:var(--text);padding:8px 12px;border-radius:8px;font-family:'Share Tech Mono',monospace;font-size:14px;width:100%;margin-bottom:8px;text-transform:uppercase}
  .verify-input:focus{outline:none;border-color:var(--purple)}
  .tag-row{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:12px}
  .tag{background:var(--bg4);border:1px solid var(--border);border-radius:20px;padding:4px 12px;font-size:12px;color:var(--muted)}
  .tag-green{border-color:var(--green);color:var(--green);background:#0d3320}
  .tag-purple{border-color:var(--purple);color:var(--purple);background:#1a0d33}
  .tag-gold{border-color:var(--gold);color:var(--gold);background:#2a1800}
  .crash-multiplier{font-size:48px;font-weight:700;font-family:'Share Tech Mono',monospace;text-align:center;margin:20px 0;color:var(--green)}
  .crash-multiplier.crashed{color:var(--red)}
  .crash-controls{display:flex;gap:12px;margin-top:16px}
  .crash-auto-box{background:var(--bg3);border:1px solid var(--border);border-radius:8px;padding:12px;margin-top:12px}
`;

// BLACKJACK GAME
function BlackjackGame({ balance, onUpdate }) {
  const [deck, setDeck] = useState(makeDeck());
  const [playerHand, setPlayerHand] = useState([]);
  const [dealerHand, setDealerHand] = useState([]);
  const [result, setResult] = useState(null);
  const [bet, setBet] = useState("");
  const [gameActive, setGameActive] = useState(false);

  const dealInitial = useCallback(() => {
    if (!bet || isNaN(bet) || parseInt(bet) <= 0 || parseInt(bet) > balance) {
      alert("Invalid bet amount");
      return;
    }
    const newDeck = makeDeck();
    const ph = [newDeck.pop(), newDeck.pop()];
    const dh = [newDeck.pop()];
    setDeck(newDeck);
    setPlayerHand(ph);
    setDealerHand(dh);
    setGameActive(true);
    setResult(null);
  }, [bet, balance]);

  const hit = useCallback(() => {
    if (!gameActive) return;
    const newDeck = [...deck];
    const card = newDeck.pop();
    const newHand = [...playerHand, card];
    setDeck(newDeck);
    setPlayerHand(newHand);
    if (handVal(newHand) > 21) endGame(newHand, dealerHand, newDeck, true);
  }, [gameActive, deck, playerHand, dealerHand]);

  const stand = useCallback(() => {
    if (!gameActive) return;
    let newDealerHand = [...dealerHand];
    let newDeck = [...deck];
    while (handVal(newDealerHand) < 17) {
      newDealerHand.push(newDeck.pop());
    }
    endGame(playerHand, newDealerHand, newDeck, false);
  }, [gameActive, dealerHand, deck, playerHand]);

  const endGame = useCallback((ph, dh, d, playerBusted) => {
    const pv = handVal(ph);
    const dv = handVal(dh);
    let res, winnings = 0;

    if (playerBusted) {
      res = "BUST - You Lose!";
      onUpdate(balance - parseInt(bet));
    } else if (dv > 21) {
      res = "Dealer Bust - You Win!";
      winnings = parseInt(bet) * 2;
      onUpdate(balance + winnings);
    } else if (pv > dv) {
      res = "You Win!";
      winnings = parseInt(bet) * 2;
      onUpdate(balance + winnings);
    } else if (pv === dv) {
      res = "Push - Tie!";
      onUpdate(balance);
    } else {
      res = "You Lose!";
      onUpdate(balance - parseInt(bet));
    }

    setResult(res);
    setGameActive(false);
    setDeck(d);
  }, [balance, bet, onUpdate]);

  const cardColor = (rank) => {
    return ["♥", "♦"].includes(rank) ? "card-r" : "card-b";
  };

  return (
    <div className="game-card">
      <h2 className="game-title">Blackjack</h2>
      <p className="game-sub">Beat the dealer without going over 21</p>

      <div className="bet-row">
        <label className="bet-label">Bet:</label>
        <input
          className="bet-input"
          type="number"
          value={bet}
          onChange={(e) => setBet(e.target.value)}
          placeholder="Amount"
          disabled={gameActive}
        />
      </div>

      <div>
        <div className="hand-label">Dealer</div>
        <div className="cards-row">
          {dealerHand.map((c, i) => (
            <div key={i} className={`card ${cardColor(c.s)}`}>
              {c.r}{c.s}
            </div>
          ))}
        </div>
      </div>

      <div>
        <div className="hand-label">Player - {gameActive ? `${handVal(playerHand)}` : "---"}</div>
        <div className="cards-row">
          {playerHand.map((c, i) => (
            <div key={i} className={`card ${cardColor(c.s)}`}>
              {c.r}{c.s}
            </div>
          ))}
        </div>
      </div>

      <div className="btn-row">
        {!gameActive ? (
          <button className="btn btn-green" onClick={dealInitial}>Deal</button>
        ) : (
          <>
            <button className="btn btn-green" onClick={hit}>Hit</button>
            <button className="btn btn-gold" onClick={stand}>Stand</button>
          </>
        )}
      </div>

      {result && <div className={`result-box result-${result.includes("Win") ? "win" : result.includes("Tie") ? "push" : "lose"}`}>{result}</div>}
    </div>
  );
}

// COINFLIP GAME
function CoinflipGame({ balance, onUpdate, onFeed }) {
  const [bet, setBet] = useState("");
  const [choice, setChoice] = useState(null);
  const [flipping, setFlipping] = useState(false);
  const [result, setResult] = useState(null);

  const flip = useCallback(() => {
    if (!bet || isNaN(bet) || parseInt(bet) <= 0 || parseInt(bet) > balance) {
      alert("Invalid bet amount");
      return;
    }
    if (!choice) {
      alert("Choose Heads or Tails");
      return;
    }

    setFlipping(true);
    setTimeout(() => {
      const outcome = Math.random() < 0.5 ? "heads" : "tails";
      const won = outcome === choice;
      const betAmount = parseInt(bet);
      const winnings = won ? betAmount * 2 : 0;

      setResult(outcome);
      setFlipping(false);
      onUpdate(balance + (won ? betAmount : -betAmount));
      onFeed(rndName(), "Coinflip", betAmount, won);
      setBet("");
      setChoice(null);
    }, 1000);
  }, [bet, choice, balance, onUpdate, onFeed]);

  return (
    <div className="game-card">
      <h2 className="game-title">Coinflip</h2>
      <p className="game-sub">50/50 chance to double your bet</p>

      <div className="bet-row">
        <label className="bet-label">Bet:</label>
        <input
          className="bet-input"
          type="number"
          value={bet}
          onChange={(e) => setBet(e.target.value)}
          placeholder="Amount"
          disabled={flipping}
        />
      </div>

      <div className="coin-area">
        <div className={`coin ${flipping ? "coin-idle spinning" : result === "heads" ? "coin-h" : result === "tails" ? "coin-t" : "coin-idle"}`}>
          {result === "heads" ? "H" : result === "tails" ? "T" : "?"}
        </div>
        <div className="side-btns">
          <button
            className={`side-btn side-btn-h ${choice === "heads" ? "chosen" : ""}`}
            onClick={() => setChoice(choice === "heads" ? null : "heads")}
            disabled={flipping}
          >
            Heads
          </button>
          <button
            className={`side-btn side-btn-t ${choice === "tails" ? "chosen" : ""}`}
            onClick={() => setChoice(choice === "tails" ? null : "tails")}
            disabled={flipping}
          >
            Tails
          </button>
        </div>
      </div>

      <button className="btn btn-green" onClick={flip} disabled={flipping || !bet || !choice}>
        {flipping ? "Flipping..." : "Flip"}
      </button>

      {result && !flipping && (
        <div className={`result-box result-${result === choice ? "win" : "lose"}`}>
          {result === choice ? "You Win!" : "You Lose!"}
        </div>
      )}
    </div>
  );
}

// ROULETTE GAME
function RouletteGame({ balance, onUpdate, onFeed }) {
  const [bet, setBet] = useState("");
  const [bets, setBets] = useState({});
  const [spinning, setSpinning] = useState(false);
  const [result, setResult] = useState(null);

  const totalBet = useMemo(() => Object.values(bets).reduce((a, b) => a + b, 0), [bets]);

  const toggleBet = useCallback((type) => {
    if (!bet || isNaN(bet) || parseInt(bet) <= 0) return;
    const amount = parseInt(bet);
    setBets(prev => ({
      ...prev,
      [type]: prev[type] ? 0 : amount
    }));
  }, [bet]);

  const spin = useCallback(() => {
    if (totalBet === 0 || totalBet > balance) {
      alert("Invalid bet");
      return;
    }

    setSpinning(true);
    setTimeout(() => {
      const number = Math.floor(Math.random() * 37);
      const color = ROULETTE_COLORS[number];
      let won = 0;

      if (bets.red && color === "red") won += bets.red * 2;
      if (bets.black && color === "black") won += bets.black * 2;
      if (bets.green && color === "green") won += bets.green * 37;
      if (bets.even && number !== 0 && number % 2 === 0) won += bets.even * 2;
      if (bets.odd && number % 2 === 1) won += bets.odd * 2;
      if (bets.low && number > 0 && number <= 18) won += bets.low * 2;
      if (bets.high && number > 18) won += bets.high * 2;

      const netWin = won - totalBet;
      setResult(number);
      setSpinning(false);
      onUpdate(balance + netWin);
      onFeed(rndName(), "Roulette", totalBet, won > totalBet);
      setBets({});
      setBet("");
    }, 2000);
  }, [bets, balance, totalBet, onUpdate, onFeed]);

  const resultColor = result !== null ? ROULETTE_COLORS[result] : "idle";

  return (
    <div className="game-card">
      <h2 className="game-title">Roulette</h2>
      <p className="game-sub">Place your bets on the wheel</p>

      <div className={`roulette-wheel rw-${resultColor}`}>
        {result !== null ? result : "?"}
      </div>

      <div className="bet-row">
        <label className="bet-label">Bet:</label>
        <input
          className="bet-input"
          type="number"
          value={bet}
          onChange={(e) => setBet(e.target.value)}
          placeholder="Amount"
          disabled={spinning}
        />
      </div>

      <div className="roulette-bets">
        <button
          className={`rbet rbet-red ${bets.red ? "sel" : ""}`}
          onClick={() => toggleBet("red")}
          disabled={spinning}
        >
          RED
        </button>
        <button
          className={`rbet rbet-black ${bets.black ? "sel" : ""}`}
          onClick={() => toggleBet("black")}
          disabled={spinning}
        >
          BLACK
        </button>
        <button
          className={`rbet rbet-green ${bets.green ? "sel" : ""}`}
          onClick={() => toggleBet("green")}
          disabled={spinning}
        >
          GREEN
        </button>
        <button
          className={`rbet rbet-even ${bets.even ? "sel" : ""}`}
          onClick={() => toggleBet("even")}
          disabled={spinning}
        >
          EVEN
        </button>
        <button
          className={`rbet rbet-odd ${bets.odd ? "sel" : ""}`}
          onClick={() => toggleBet("odd")}
          disabled={spinning}
        >
          ODD
        </button>
        <button
          className={`rbet rbet-low ${bets.low ? "sel" : ""}`}
          onClick={() => toggleBet("low")}
          disabled={spinning}
        >
          1-18
        </button>
        <button
          className={`rbet rbet-high ${bets.high ? "sel" : ""}`}
          onClick={() => toggleBet("high")}
          disabled={spinning}
        >
          19-36
        </button>
      </div>

      <div className="bet-row" style={{ marginTop: "12px", justifyContent: "space-between" }}>
        <span>Total Bet: <span style={{ color: "var(--gold)" }}>${totalBet}</span></span>
        <button className="btn btn-gold" onClick={spin} disabled={spinning || totalBet === 0}>
          {spinning ? "Spinning..." : "Spin"}
        </button>
      </div>

      {result !== null && !spinning && (
        <div className={`result-box result-${result === 0 ? "info" : "win"}`}>
          Landed on {result} ({ROULETTE_COLORS[result]})
        </div>
      )}
    </div>
  );
}

// CRASH GAME
function CrashGame({ balance, onUpdate, onFeed }) {
  const [bet, setBet] = useState("");
  const [playing, setPlaying] = useState(false);
  const [multiplier, setMultiplier] = useState(1.0);
  const [crashed, setCrashed] = useState(false);
  const [cashed, setCashed] = useState(false);
  const crashPointRef = useRef(Math.random() * 4 + 1);
  const multIntervalRef = useRef(null);

  const start = useCallback(() => {
    if (!bet || isNaN(bet) || parseInt(bet) <= 0 || parseInt(bet) > balance) {
      alert("Invalid bet");
      return;
    }

    setPlaying(true);
    setMultiplier(1.0);
    setCrashed(false);
    setCashed(false);
    crashPointRef.current = Math.random() * 4 + 1;

    multIntervalRef.current = setInterval(() => {
      setMultiplier(m => {
        const newMult = m + 0.1;
        if (newMult >= crashPointRef.current) {
          clearInterval(multIntervalRef.current);
          setCrashed(true);
          setPlaying(false);
          onUpdate(balance - parseInt(bet));
          onFeed(rndName(), "Crash", parseInt(bet), false);
          return crashPointRef.current;
        }
        return newMult;
      });
    }, 100);
  }, [bet, balance, onUpdate, onFeed]);

  const cashOut = useCallback(() => {
    if (!playing) return;
    clearInterval(multIntervalRef.current);
    setPlaying(false);
    setCashed(true);
    const winnings = parseInt(bet) * multiplier;
    onUpdate(balance + winnings);
    onFeed(rndName(), "Crash", parseInt(bet), true);
  }, [playing, bet, multiplier, balance, onUpdate, onFeed]);

  return (
    <div className="game-card">
      <h2 className="game-title">Crash</h2>
      <p className="game-sub">Cash out before it crashes!</p>

      <div className="bet-row">
        <label className="bet-label">Bet:</label>
        <input
          className="bet-input"
          type="number"
          value={bet}
          onChange={(e) => setBet(e.target.value)}
          placeholder="Amount"
          disabled={playing}
        />
      </div>

      <div className={`crash-multiplier ${crashed ? "crashed" : ""}`}>
        {multiplier.toFixed(2)}x
      </div>

      <div className="crash-controls">
        {!playing ? (
          <button className="btn btn-green" onClick={start} disabled={playing || cashed}>
            {cashed ? "Next Round" : "Start"}
          </button>
        ) : (
          <button className="btn btn-gold" onClick={cashOut}>
            Cash Out: ${(parseInt(bet || 0) * multiplier).toFixed(2)}
          </button>
        )}
      </div>

      {crashed && (
        <div className="result-box result-lose">
          Crashed at {multiplier.toFixed(2)}x - You Lose!
        </div>
      )}

      {cashed && !playing && (
        <div className="result-box result-win">
          Cashed out at {multiplier.toFixed(2)}x - Won ${(parseInt(bet) * multiplier).toFixed(2)}!
        </div>
      )}
    </div>
  );
}

// FEED COMPONENT
function Feed({ items }) {
  return (
    <div className="sidebar">
      <div className="section-title">Live Feed</div>
      {items.map(item => (
        <div key={item.id} className="feed-item fade-in">
          <div>
            <div className="feed-name">{item.name}</div>
            <div className="feed-game">{item.game}</div>
          </div>
          <div className={item.won ? "feed-win" : "feed-lose"}>
            {item.won ? "+" : "-"}${item.amount}
          </div>
        </div>
      ))}
    </div>
  );
}

// MAIN APP
export default function LiveMCasino() {
  const [page, setPage] = useState("home");
  const [balance, setBalance] = useState(10000);
  const [feed, setFeed] = useState([]);
  const [txHistory, setTxHistory] = useState([]);
  const [username, setUsername] = useState(null);
  const [pendingDeposits, setPendingDeposits] = useState([]);
  const feedRef = useRef([]);

  const addFeed = useCallback((name, game, amount, won) => {
    const item = { id: Date.now() + Math.random(), name, game, amount, won };
    feedRef.current = [item, ...feedRef.current].slice(0, 30);
    setFeed([...feedRef.current]);
  }, []);

  const addTx = useCallback((label, amount, plus) => {
    setTxHistory(h => [{t: label, a: amount, plus, ts: new Date().toLocaleTimeString()}, ...h].slice(0, 50));
  }, []);

  useEffect(() => {
    const iv = setInterval(() => {
      addFeed(rndName(), rndGame(), rndAmt(), Math.random() > 0.5);
    }, 3000);
    return () => clearInterval(iv);
  }, [addFeed]);

  useEffect(() => {
    if (!username) setUsername(rndName());
  }, [username]);

  const handleBalanceUpdate = useCallback((newBalance) => {
    setBalance(newBalance);
  }, []);

  const pageContent = () => {
    switch (page) {
      case "blackjack":
        return <BlackjackGame balance={balance} onUpdate={handleBalanceUpdate} />;
      case "coinflip":
        return <CoinflipGame balance={balance} onUpdate={handleBalanceUpdate} onFeed={addFeed} />;
      case "roulette":
        return <RouletteGame balance={balance} onUpdate={handleBalanceUpdate} onFeed={addFeed} />;
      case "crash":
        return <CrashGame balance={balance} onUpdate={handleBalanceUpdate} onFeed={addFeed} />;
      case "wallet":
        return (
          <div className="game-card">
            <h2 className="game-title">Wallet</h2>
            <div className="wallet-grid">
              <div className="wcard">
                <div className="wcard-label">Balance</div>
                <div className="wcard-val">${balance}</div>
              </div>
              <div className="wcard">
                <div className="wcard-label">Pending</div>
                <div className="wcard-val">${pendingDeposits.reduce((a, b) => a + b.amount, 0)}</div>
              </div>
            </div>
            <div className="section-title">Transaction History</div>
            {txHistory.map((tx, i) => (
              <div key={i} className="tx-row">
                <span>{tx.t}</span>
                <span className={tx.plus ? "tx-plus" : "tx-minus"}>{tx.plus ? "+" : "-"}${tx.a}</span>
                <span style={{ color: "var(--muted)", fontSize: "11px" }}>{tx.ts}</span>
              </div>
            ))}
          </div>
        );
      default:
        return (
          <div>
            <h1 className="hero-title">CASINO</h1>
            <p className="hero-sub">Play. Win. Repeat.</p>
            <div className="games-grid">
              {GAMES_LIST.map(g => (
                <div key={g} className="game-thumb" onClick={() => setPage(g.toLowerCase())}>
                  <h3>{g}</h3>
                  <p style={{ fontSize: "12px", color: "var(--muted)", marginTop: "8px" }}>
                    {g === "Blackjack" && "Beat the dealer"}
                    {g === "Coinflip" && "50/50 chance"}
                    {g === "Roulette" && "Spin the wheel"}
                    {g === "Crash" && "Cash out in time"}
                  </p>
                </div>
              ))}
            </div>
            <div className="stats-row">
              <div className="stat">
                <div className="stat-val">${balance}</div>
                <div className="stat-label">Balance</div>
              </div>
              <div className="stat">
                <div className="stat-val">{feed.length}</div>
                <div className="stat-label">Feed</div>
              </div>
            </div>
          </div>
        );
    }
  };

  return (
    <div>
      <style>{css}</style>
      <div className="app">
        <nav className="nav">
          <div className="logo">LIVE CASINO</div>
          <button className={`nav-btn ${page === "home" ? "active" : ""}`} onClick={() => setPage("home")}>Home</button>
          <button className={`nav-btn ${page === "blackjack" ? "active" : ""}`} onClick={() => setPage("blackjack")}>Blackjack</button>
          <button className={`nav-btn ${page === "coinflip" ? "active" : ""}`} onClick={() => setPage("coinflip")}>Coinflip</button>
          <button className={`nav-btn ${page === "roulette" ? "active" : ""}`} onClick={() => setPage("roulette")}>Roulette</button>
          <button className={`nav-btn ${page === "crash" ? "active" : ""}`} onClick={() => setPage("crash")}>Crash</button>
          <button className={`nav-btn ${page === "wallet" ? "active" : ""}`} onClick={() => setPage("wallet")}>Wallet</button>
          <div className="balance-pill">${balance}</div>
        </nav>

        <div className="main">
          <div className="content">
            {pageContent()}
          </div>
          <Feed items={feed} />
        </div>
      </div>
    </div>
  );
}
```
