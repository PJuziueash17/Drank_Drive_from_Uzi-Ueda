import { useState, useEffect, useRef } from “react”;

// ============================================================
// CONSTANTS & CONFIG
// ============================================================
const SCORE_TO_METERS = 1;
const WARNING_THRESHOLD = 8750;
const REF_BEER_SCORE = 350 * 5;

const PRESETS = [
{ name: “ビール”, volume: 350, abv: 5, icon: “🍺” },
{ name: “ハイボール”, volume: 350, abv: 7, icon: “🥃” },
{ name: “レモンサワー”, volume: 350, abv: 5, icon: “🍋” },
{ name: “ワイン”, volume: 125, abv: 12, icon: “🍷” },
{ name: “日本酒”, volume: 180, abv: 15, icon: “🍶” },
{ name: “焼酎”, volume: 100, abv: 25, icon: “🫗” },
{ name: “テキーラ”, volume: 30, abv: 40, icon: “🌵” },
{ name: “ウイスキー”, volume: 30, abv: 40, icon: “🥃” },
];

const PRESET_DEST = [
{ name: “渋谷駅”, distance: 5000, emoji: “🏙️” },
{ name: “東京タワー”, distance: 8000, emoji: “🗼” },
{ name: “横浜中華街”, distance: 15000, emoji: “🥟” },
{ name: “鎌倉大仏”, distance: 25000, emoji: “🗿” },
{ name: “富士山五合目”, distance: 50000, emoji: “🗻” },
{ name: “大阪道頓堀”, distance: 250000, emoji: “🏯” },
];

const RANKS = [
{ min: 0, label: “初心者ドライバー”, comment: “まだウォーミングアップ” },
{ min: 2000, label: “酔拳ドライバー”, comment: “エンジンかかってきた！” },
{ min: 5000, label: “泥酔レーサー”, comment: “今日はかなり飛ばしてる！” },
{ min: 10000, label: “暴走酒豪”, comment: “止まらねぇ…！” },
{ min: 20000, label: “伝説の泥酔王”, comment: “もはや伝説級…！” },
];

// ============================================================
// CALC
// ============================================================
function calcScore(vol, abv, cups = 1) { return Math.round(vol * abv * cups * 100) / 100; }
function s2m(s) { return s * SCORE_TO_METERS; }
function m2s(m) { return m / SCORE_TO_METERS; }
function getRank(d) { let r = RANKS[0]; for (let i = RANKS.length - 1; i >= 0; i–) { if (d >= RANKS[i].min) { r = RANKS[i]; break; } } return r; }
function fmtD(m) { return m >= 1000 ? `${(m / 1000).toFixed(1)}km` : `${Math.round(m)}m`; }
function fmtP(r) { return `${Math.min(Math.round(r * 100), 100)}%`; }

// ============================================================
// STORAGE
// ============================================================
async function ld(k, fb) { try { const r = await window.storage.get(k); return r ? JSON.parse(r.value) : fb; } catch { return fb; } }
async function sv(k, v) { try { await window.storage.set(k, JSON.stringify(v)); } catch (e) { console.error(“Save:”, e); } }

// ============================================================
// STYLES
// ============================================================
const font = `'Noto Sans JP','Hiragino Sans',sans-serif`;
const fontD = `'Shippori Mincho B1',serif`;
const C = { bg:”#0c0c12”,card:”#161620”,acc:”#ff8c1a”,accDim:”#cc6a00”,accGlow:“rgba(255,140,26,0.12)”,text:”#e8e4de”,dim:”#7a7670”,danger:”#e85050”,success:”#50c878”,border:”#28283a”,overlay:“rgba(0,0,0,0.75)”,route:”#3a3a5a”,routeAct:”#ff8c1a”,routeRem:”#4a4a6a” };
const B = { padding:“14px 24px”,borderRadius:14,border:“none”,fontFamily:font,fontSize:16,fontWeight:700,cursor:“pointer”,transition:“all 0.2s”,display:“inline-flex”,alignItems:“center”,justifyContent:“center”,gap:8 };

// ============================================================
// APP
// ============================================================
export default function App() {
const [ready, setReady] = useState(false);
const [ageOk, setAgeOk] = useState(false);
const [scr, setScr] = useState(“home”);
const [sess, setSess] = useState({ drinks: [], startedAt: null });
const [hist, setHist] = useState([]);
const [dest, setDest] = useState(null);
const [warn, setWarn] = useState(false);
const [vSess, setVSess] = useState(null);
const [arrAnim, setArrAnim] = useState(null);
const [sett, setSett] = useState({ showRef: true });
const pDist = useRef(0);

useEffect(() => { (async () => {
setAgeOk(await ld(“age”, false)); setHist(await ld(“hist”, [])); setSess(await ld(“sess”, { drinks: [], startedAt: null }));
setDest(await ld(“dest”, null)); setSett(await ld(“sett”, { showRef: true })); setReady(true);
})(); }, []);

useEffect(() => { if (ready) sv(“sess”, sess); }, [sess, ready]);
useEffect(() => { if (ready) sv(“hist”, hist); }, [hist, ready]);
useEffect(() => { if (ready) sv(“dest”, dest); }, [dest, ready]);
useEffect(() => { if (ready) sv(“sett”, sett); }, [sett, ready]);

const tScore = sess.drinks.reduce((a, d) => a + d.score, 0);
const tDist = s2m(tScore);
const tCups = sess.drinks.reduce((a, d) => a + d.cups, 0);
const rDist = dest ? dest.distance : 0;
const rem = rDist ? Math.max(rDist - tDist, 0) : 0;
const pPct = rDist ? Math.min(tDist / rDist, 1) : 0;
const arrived = rDist > 0 && tDist >= rDist;

useEffect(() => {
if (arrived && pDist.current < rDist) { setArrAnim(dest); setTimeout(() => setArrAnim(null), 3500); }
pDist.current = tDist;
}, [tDist, arrived]);

const ageConfirm = async () => { setAgeOk(true); await sv(“age”, true); };

const addDrink = (dk) => {
const sc = calcScore(dk.volume, dk.abv, dk.cups);
const nd = { …dk, score: sc, id: Date.now() };
const u = { …sess, drinks: […sess.drinks, nd], startedAt: sess.startedAt || new Date().toISOString() };
setSess(u);
if (u.drinks.reduce((a, d) => a + d.score, 0) >= WARNING_THRESHOLD) setWarn(true);
setScr(“home”);
};

const endSess = () => {
if (!sess.drinks.length) return;
const s = { …sess, endedAt: new Date().toISOString(), totalScore: tScore, totalDist: tDist, totalCups: tCups, destination: dest ? { name: dest.name, distance: dest.distance, emoji: dest.emoji } : null, arrived, id: Date.now() };
setHist(p => [s, …p]); setVSess(s); setSess({ drinks: [], startedAt: null }); setScr(“result”);
};

if (!ready) return <Splash />;
if (!ageOk) return <AgeGate onOk={ageConfirm} />;

const p = { tScore, tDist, tCups, sess, dest, rDist, rem, pPct, arrived, sett };

return (
<div style={{ fontFamily:font,background:C.bg,color:C.text,minHeight:“100vh”,maxWidth:480,margin:“0 auto”,position:“relative” }}>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700;900&family=Shippori+Mincho+B1:wght@700&display=swap" rel="stylesheet" />
{scr===“home” && <Home {…p} nav={setScr} endSess={endSess} />}
{scr===“input” && <DrinkIn onAdd={addDrink} onBack={()=>setScr(“home”)} />}
{scr===“dest” && <DestPick cur={dest} onSet={d=>{setDest(d);setScr(“home”)}} onBack={()=>setScr(“home”)} />}
{scr===“map” && <RouteMap {…p} nav={setScr} />}
{scr===“result” && <Result sess={vSess||{…sess,totalScore:tScore,totalDist:tDist,totalCups:tCups,destination:dest?{name:dest.name,distance:dest.distance,emoji:dest.emoji}:null,arrived}} onBack={()=>{setVSess(null);setScr(“home”)}} />}
{scr===“history” && <Hist list={hist} onBack={()=>setScr(“home”)} onView={s=>{setVSess(s);setScr(“result”)}} onDel={id=>setHist(h=>h.filter(s=>s.id!==id))} />}
{scr===“settings” && <Setts val={sett} set={setSett} onBack={()=>setScr(“home”)} />}
{warn && <WarnPop onClose={()=>setWarn(false)} />}
{arrAnim && <ArrFx d={arrAnim} />}
<SafetyBar />
</div>
);
}

// ============================================================
function Splash() {
return <div style={{fontFamily:font,background:C.bg,color:C.text,minHeight:“100vh”,display:“flex”,alignItems:“center”,justifyContent:“center”,flexDirection:“column”}}>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700;900&family=Shippori+Mincho+B1:wght@700&display=swap" rel="stylesheet" />
<div style={{fontSize:48}}>🏎️</div><div style={{fontSize:16,color:C.dim,marginTop:12}}>読み込み中…</div></div>;
}

function AgeGate({onOk}) {
return <div style={{fontFamily:font,background:C.bg,color:C.text,minHeight:“100vh”,display:“flex”,flexDirection:“column”,alignItems:“center”,justifyContent:“center”,padding:28,textAlign:“center”}}>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700;900&family=Shippori+Mincho+B1:wght@700&display=swap" rel="stylesheet" />
<div style={{fontSize:64,marginBottom:20}}>🏎️</div>
<h1 style={{fontFamily:fontD,fontSize:30,color:C.acc,marginBottom:4}}>泥酔ドライブ</h1>
<p style={{fontSize:13,color:C.dim,marginBottom:28}}>Drunk Drive</p>
<div style={{background:C.card,borderRadius:20,padding:24,maxWidth:380,width:“100%”,border:`1px solid ${C.border}`}}>
<h2 style={{fontSize:19,marginBottom:16,fontWeight:700}}>年齢確認</h2>
<p style={{fontSize:14,lineHeight:1.8,color:C.dim,marginBottom:14}}>このアプリは<strong style={{color:C.acc}}>20歳以上</strong>の方を対象としています。20歳未満の方はご利用いただけません。</p>
<div style={{background:“rgba(232,80,80,0.07)”,borderRadius:12,padding:14,marginBottom:20,border:“1px solid rgba(232,80,80,0.18)”}}>
<p style={{fontSize:12,lineHeight:1.7,color:”#e8a0a0”,margin:0}}>⚠️ このアプリは飲酒を推奨するものではありません。飲酒量の記録とエンターテインメントを目的としています。健康指標として利用しないでください。飲酒運転は法律で禁止されています。</p>
</div>
<button onClick={onOk} style={{…B,width:“100%”,background:`linear-gradient(135deg,${C.acc},${C.accDim})`,color:”#1a1a24”,fontSize:16}}>20歳以上です・同意して始める</button>
</div></div>;
}

// ============================================================
// HOME
// ============================================================
function Home({tScore,tDist,tCups,sess,dest,rDist,rem,pPct,arrived,nav,endSess}) {
return <div style={{padding:“20px 18px 110px”}}>
<div style={{display:“flex”,alignItems:“center”,justifyContent:“space-between”,marginBottom:22}}>
<div><h1 style={{fontFamily:fontD,fontSize:26,color:C.acc,margin:0}}>泥酔ドライブ</h1><p style={{fontSize:11,color:C.dim,margin:0}}>Drunk Drive</p></div>
<div style={{display:“flex”,gap:8}}><SB onClick={()=>nav(“history”)}>📖</SB><SB onClick={()=>nav(“settings”)}>⚙️</SB></div>
</div>

```
{/* Destination */}
<div onClick={()=>nav("dest")} style={{background:C.card,borderRadius:18,padding:18,marginBottom:14,border:`1px solid ${C.border}`,cursor:"pointer"}}>
  {dest ? <div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
    <div><div style={{fontSize:11,color:C.dim,marginBottom:4}}>🎯 目的地</div><div style={{fontSize:18,fontWeight:900}}>{dest.emoji||"📍"} {dest.name}</div><div style={{fontSize:12,color:C.dim}}>{fmtD(dest.distance)}</div></div>
    <div style={{fontSize:12,color:C.acc}}>変更 →</div>
  </div> : <div style={{textAlign:"center",padding:8}}><div style={{fontSize:14,color:C.acc,fontWeight:700}}>🎯 目的地を設定する</div><div style={{fontSize:12,color:C.dim,marginTop:4}}>タップして目的地を選ぼう</div></div>}
</div>

{/* Progress */}
{dest && <div style={{background:`linear-gradient(135deg,#1a1610,${C.card})`,borderRadius:18,padding:18,marginBottom:14,border:`1px solid ${C.border}`,boxShadow:`0 4px 20px ${C.accGlow}`}}>
  {arrived ? <div style={{textAlign:"center",padding:8}}><div style={{fontSize:36}}>🎉</div><div style={{fontFamily:fontD,fontSize:22,color:C.acc,marginTop:8}}>到着！</div><div style={{fontSize:13,color:C.dim}}>{dest.name}に到達しました</div></div>
  : <>
    <div style={{display:"flex",justifyContent:"space-between",marginBottom:12}}>
      <St l="進行距離" v={fmtD(tDist)} hi /><St l="残距離" v={fmtD(rem)} /><St l="進捗率" v={fmtP(pPct)} hi />
    </div>
    <PBar pct={pPct} />
    <div style={{display:"flex",justifyContent:"space-between",fontSize:10,color:C.dim,marginTop:4}}><span>🏎️ 出発</span><span>{dest.emoji||"📍"} {dest.name}</span></div>
  </>}
</div>}

{!dest && sess.drinks.length > 0 && <div style={{background:C.card,borderRadius:18,padding:18,marginBottom:14,border:`1px solid ${C.border}`}}>
  <div style={{display:"flex",gap:12}}><St l="スコア" v={`${tScore.toLocaleString()}pt`} hi /><St l="距離換算" v={fmtD(tDist)} /><St l="杯数" v={`${tCups}杯`} /></div>
</div>}

{/* Drinks */}
{sess.drinks.length > 0 && <div style={{background:C.card,borderRadius:16,padding:14,marginBottom:14,border:`1px solid ${C.border}`}}>
  <div style={{fontSize:12,color:C.dim,marginBottom:8}}>🍺 今日飲んだもの</div>
  {sess.drinks.map((d,i)=><div key={d.id} style={{display:"flex",justifyContent:"space-between",padding:"6px 0",borderBottom:i<sess.drinks.length-1?`1px solid ${C.border}`:"none",fontSize:14}}>
    <span>{d.name} ×{d.cups}</span><span style={{color:C.acc}}>{d.score.toLocaleString()}pt</span></div>)}
</div>}

{/* Actions */}
<div style={{display:"flex",flexDirection:"column",gap:10}}>
  <button onClick={()=>nav("input")} style={{...B,width:"100%",padding:"16px",background:`linear-gradient(135deg,${C.acc},${C.accDim})`,color:"#1a1a24",fontSize:17,fontWeight:900,borderRadius:16}}>🍺 飲んだものを追加</button>
  <div style={{display:"flex",gap:10}}>
    {dest && <button onClick={()=>nav("map")} style={{...B,flex:1,background:C.card,color:C.text,border:`1px solid ${C.border}`,borderRadius:14,fontSize:14}}>🗺️ ルート</button>}
    <button onClick={endSess} disabled={!sess.drinks.length} style={{...B,flex:1,fontSize:14,borderRadius:14,background:sess.drinks.length?"rgba(232,80,80,0.12)":C.card,color:sess.drinks.length?"#e8a0a0":C.dim,border:`1px solid ${sess.drinks.length?"rgba(232,80,80,0.25)":C.border}`,opacity:sess.drinks.length?1:0.5}}>🏁 旅を終える</button>
  </div>
</div>
```

  </div>;
}

// ============================================================
// DEST PICKER
// ============================================================
function DestPick({cur,onSet,onBack}) {
const [mode,setMode]=useState(“preset”);
const [cn,setCn]=useState(””); const [cd,setCd]=useState(””); const [err,setErr]=useState(””);
const inp={width:“100%”,padding:“14px 16px”,borderRadius:12,border:`1px solid ${C.border}`,background:”#101018”,color:C.text,fontSize:16,fontFamily:font,outline:“none”,boxSizing:“border-box”};

return <div style={{padding:“20px 18px 110px”}}>
<div style={{display:“flex”,alignItems:“center”,gap:10,marginBottom:20}}><SB onClick={onBack}>←</SB><h2 style={{fontSize:19,fontWeight:700,margin:0}}>🎯 目的地を設定</h2></div>

```
{cur && <div style={{background:C.card,borderRadius:14,padding:14,marginBottom:16,border:`1px solid ${C.border}`}}>
  <div style={{fontSize:12,color:C.dim}}>現在の目的地</div>
  <div style={{fontSize:16,fontWeight:700,marginTop:4}}>{cur.emoji} {cur.name}（{fmtD(cur.distance)}）</div>
  <button onClick={()=>onSet(null)} style={{...B,padding:"8px 14px",fontSize:12,background:"rgba(232,80,80,0.1)",color:"#e8a0a0",border:"1px solid rgba(232,80,80,0.2)",borderRadius:10,marginTop:10}}>クリア</button>
</div>}

<div style={{display:"flex",gap:8,marginBottom:18}}>
  {[["preset","おすすめ"],["custom","カスタム"]].map(([k,l])=>
    <button key={k} onClick={()=>setMode(k)} style={{...B,flex:1,padding:"10px",fontSize:14,background:mode===k?C.accGlow:C.card,color:mode===k?C.acc:C.dim,border:`1px solid ${mode===k?C.acc:C.border}`,borderRadius:12}}>{l}</button>)}
</div>

{mode==="preset" && <div style={{display:"flex",flexDirection:"column",gap:10}}>
  {PRESET_DEST.map(p=><button key={p.name} onClick={()=>onSet(p)} style={{...B,padding:"16px 18px",justifyContent:"space-between",background:C.card,color:C.text,border:`1px solid ${C.border}`,borderRadius:14,width:"100%"}}>
    <span style={{fontSize:15}}>{p.emoji} {p.name}</span><span style={{fontSize:13,color:C.acc}}>{fmtD(p.distance)}</span></button>)}
</div>}

{mode==="custom" && <div style={{display:"flex",flexDirection:"column",gap:14}}>
  <div><label style={{fontSize:13,color:C.dim,display:"block",marginBottom:6}}>目的地名</label><input value={cn} onChange={e=>{setCn(e.target.value);setErr("")}} placeholder="例: 友達の家" style={inp} /></div>
  <div><label style={{fontSize:13,color:C.dim,display:"block",marginBottom:6}}>距離 (km)</label><input type="number" value={cd} onChange={e=>{setCd(e.target.value);setErr("")}} placeholder="例: 10" style={inp} inputMode="decimal" /></div>
  {err && <div style={{fontSize:13,color:C.danger}}>{err}</div>}
  <button onClick={()=>{if(!cn.trim()){setErr("名前を入力");return}const d=parseFloat(cd);if(!d||d<=0){setErr("正しい距離を入力");return}onSet({name:cn.trim(),distance:Math.round(d*1000),emoji:"📍"})}} style={{...B,width:"100%",background:`linear-gradient(135deg,${C.acc},${C.accDim})`,color:"#1a1a24",fontSize:16,borderRadius:14}}>設定する</button>
</div>}

<div style={{marginTop:20,fontSize:11,color:C.dim,lineHeight:1.6,textAlign:"center"}}>※ 距離はエンタメ上の仮想距離です。実際の移動距離とは異なります。</div>
```

  </div>;
}

// ============================================================
// ROUTE MAP
// ============================================================
function RouteMap({tDist,rDist,rem,pPct,arrived,dest,sett,nav}) {
const dS=rem>0?m2s(rem):0; const dB=dS/REF_BEER_SCORE;
const W=12;
const pts=Array.from({length:W+1},(_,i)=>{const t=i/W;return{x:40+t*300,y:220-Math.sin(t*Math.PI)*120+Math.sin(t*Math.PI*2.5)*30};});
const pI=Math.min(Math.floor(pPct*W),W);
const pF=(pPct*W)-Math.floor(pPct*W);
let cx,cy;
if(arrived||pI>=W){cx=pts[W].x;cy=pts[W].y;}else{const a=pts[pI],b=pts[Math.min(pI+1,W)];cx=a.x+(b.x-a.x)*pF;cy=a.y+(b.y-a.y)*pF;}
const pathD=pts.map((p,i)=>`${i===0?"M":"L"}${p.x},${p.y}`).join(” “);
const aP=[…pts.slice(0,pI+1)];if(pI<W)aP.push({x:cx,y:cy});
const actD=aP.map((p,i)=>`${i===0?"M":"L"}${p.x},${p.y}`).join(” “);

return <div style={{padding:“20px 18px 110px”}}>
<div style={{display:“flex”,alignItems:“center”,gap:10,marginBottom:18}}><SB onClick={()=>nav(“home”)}>←</SB><h2 style={{fontSize:19,fontWeight:700,margin:0}}>🗺️ ルートマップ</h2></div>

```
<div style={{background:C.card,borderRadius:20,padding:4,marginBottom:16,border:`1px solid ${C.border}`,overflow:"hidden"}}>
  <svg viewBox="0 0 380 280" style={{width:"100%",display:"block"}}>
    <defs>
      <filter id="gl"><feGaussianBlur stdDeviation="4" result="g"/><feMerge><feMergeNode in="g"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
      <linearGradient id="rg" x1="0%" y1="0%" x2="100%" y2="0%"><stop offset="0%" stopColor={C.acc}/><stop offset="100%" stopColor="#ffaa44"/></linearGradient>
    </defs>
    {Array.from({length:30},(_,i)=><circle key={i} cx={20+Math.random()*340} cy={20+Math.random()*240} r={1} fill="#333348" opacity={0.5}/>)}
    <path d={pathD} fill="none" stroke={C.route} strokeWidth="6" strokeLinecap="round" strokeDasharray="2 8"/>
    <path d={actD} fill="none" stroke="url(#rg)" strokeWidth="6" strokeLinecap="round" filter="url(#gl)"/>
    <circle cx={pts[0].x} cy={pts[0].y} r="14" fill={C.card} stroke={C.acc} strokeWidth="2"/><text x={pts[0].x} y={pts[0].y+5} textAnchor="middle" fontSize="14">🏎️</text><text x={pts[0].x} y={pts[0].y+30} textAnchor="middle" fontSize="9" fill={C.dim}>出発</text>
    <circle cx={pts[W].x} cy={pts[W].y} r="14" fill={arrived?C.acc:C.card} stroke={arrived?C.acc:C.border} strokeWidth="2"/><text x={pts[W].x} y={pts[W].y+5} textAnchor="middle" fontSize="14">{dest?.emoji||"📍"}</text><text x={pts[W].x} y={pts[W].y+30} textAnchor="middle" fontSize="9" fill={C.dim}>{dest?.name||"目的地"}</text>
    {!arrived&&tDist>0&&<><circle cx={cx} cy={cy} r="18" fill="none" stroke={C.acc} strokeWidth="1.5" opacity="0.4"><animate attributeName="r" values="14;22;14" dur="2s" repeatCount="indefinite"/><animate attributeName="opacity" values="0.5;0.1;0.5" dur="2s" repeatCount="indefinite"/></circle><circle cx={cx} cy={cy} r="10" fill={C.acc} stroke="#fff" strokeWidth="2"/><text x={cx} y={cy-18} textAnchor="middle" fontSize="10" fill={C.acc} fontWeight="bold">{fmtP(pPct)}</text></>}
    {[0.25,0.5,0.75].map(t=>{const idx=Math.floor(t*W);const p=pts[idx];return<text key={t} x={p.x} y={p.y-14} textAnchor="middle" fontSize="8" fill={C.dim}>{fmtD(rDist*t)}</text>;})}
  </svg>
</div>

<div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:14}}>
  <SC l="総距離" v={fmtD(rDist)}/><SC l="進行距離" v={fmtD(tDist)} a/><SC l="残距離" v={arrived?"到着！":fmtD(rem)}/><SC l="進捗率" v={arrived?"100%":fmtP(pPct)} a/>
</div>

{sett.showRef&&!arrived&&rem>0&&<div style={{background:"rgba(255,140,26,0.06)",borderRadius:14,padding:16,marginBottom:14,border:"1px solid rgba(255,140,26,0.15)"}}>
  <div style={{fontSize:13,fontWeight:700,marginBottom:8,color:C.acc}}>📊 距離差分の参考値</div>
  <div style={{fontSize:14,marginBottom:6}}>残り <strong style={{color:C.acc}}>{fmtD(rem)}</strong> 相当のスコア: <strong>{Math.round(dS).toLocaleString()}pt</strong></div>
  <div style={{fontSize:13,color:C.dim,marginBottom:10}}>ビール中ジョッキ換算で約 <strong style={{color:C.text}}>{dB.toFixed(1)}杯</strong> 相当</div>
  <div style={{fontSize:10,color:C.dim,lineHeight:1.6,padding:"8px 0 0",borderTop:`1px solid ${C.border}`}}>⚠️ この表示はエンタメ上の距離換算です。追加摂取の推奨ではありません。飲酒を促す目的の表示ではありません。</div>
</div>}

{arrived&&<div style={{background:"rgba(80,200,120,0.08)",borderRadius:14,padding:20,marginBottom:14,border:"1px solid rgba(80,200,120,0.2)",textAlign:"center"}}>
  <div style={{fontSize:40}}>🎉</div><div style={{fontFamily:fontD,fontSize:22,color:C.success,marginTop:8}}>{dest?.name}に到着！</div></div>}

<div style={{display:"flex",gap:10}}>
  <button onClick={()=>nav("input")} style={{...B,flex:1,background:`linear-gradient(135deg,${C.acc},${C.accDim})`,color:"#1a1a24",fontSize:14,borderRadius:14}}>🍺 追加</button>
  <button onClick={()=>nav("dest")} style={{...B,flex:1,background:C.card,color:C.text,border:`1px solid ${C.border}`,borderRadius:14,fontSize:14}}>🎯 目的地変更</button>
</div>
```

  </div>;
}

// ============================================================
// DRINK INPUT
// ============================================================
function DrinkIn({onAdd,onBack}) {
const [nm,setNm]=useState(””);const [vol,setVol]=useState(””);const [abv,setAbv]=useState(””);const [cups,setCups]=useState(“1”);const [memo,setMemo]=useState(””);const [errs,setErrs]=useState({});
const pk=p=>{setNm(p.name);setVol(String(p.volume));setAbv(String(p.abv));setErrs({});};
const validate=()=>{const e={};if(!nm.trim())e.nm=“飲み物名を入力”;const v=parseFloat(vol),a=parseFloat(abv),c=parseInt(cups);if(!v||v<=0||v>10000)e.vol=“正しい容量を入力”;if(!a||a<=0||a>100)e.abv=“正しい度数を入力”;if(!c||c<=0||c>50)e.cups=“正しい杯数を入力”;setErrs(e);return!Object.keys(e).length;};
const submit=()=>{if(!validate())return;onAdd({name:nm.trim(),volume:parseFloat(vol),abv:parseFloat(abv),cups:parseInt(cups),memo:memo.trim(),time:new Date().toISOString()});};
const prev=(vol&&abv&&cups)?calcScore(parseFloat(vol)||0,parseFloat(abv)||0,parseInt(cups)||1):0;
const inp={width:“100%”,padding:“14px 16px”,borderRadius:12,border:`1px solid ${C.border}`,background:”#101018”,color:C.text,fontSize:16,fontFamily:font,outline:“none”,boxSizing:“border-box”};

return <div style={{padding:“20px 18px 110px”}}>
<div style={{display:“flex”,alignItems:“center”,gap:10,marginBottom:20}}><SB onClick={onBack}>←</SB><h2 style={{fontSize:19,fontWeight:700,margin:0}}>🍺 飲んだものを追加</h2></div>
<div style={{marginBottom:20}}><div style={{fontSize:12,color:C.dim,marginBottom:8}}>プリセット</div>
<div style={{display:“flex”,flexWrap:“wrap”,gap:8}}>{PRESETS.map(p=><button key={p.name} onClick={()=>pk(p)} style={{…B,padding:“9px 13px”,fontSize:13,background:nm===p.name?C.accGlow:C.card,color:nm===p.name?C.acc:C.text,border:`1px solid ${nm===p.name?C.acc:C.border}`,borderRadius:11}}>{p.icon} {p.name}</button>)}</div>
</div>
<div style={{display:“flex”,flexDirection:“column”,gap:14}}>
<Fl l="飲み物名" e={errs.nm}><input value={nm} onChange={e=>setNm(e.target.value)} placeholder=“例: ビール” style={inp}/></Fl>
<div style={{display:“flex”,gap:10}}>
<Fl l="容量(ml)" e={errs.vol} f={1}><input type=“number” value={vol} onChange={e=>setVol(e.target.value)} placeholder=“350” style={inp} inputMode=“decimal”/></Fl>
<Fl l="度数(%)" e={errs.abv} f={1}><input type=“number” value={abv} onChange={e=>setAbv(e.target.value)} placeholder=“5” style={inp} inputMode=“decimal”/></Fl>
<Fl l="杯数" e={errs.cups} f={0.6}><input type=“number” value={cups} onChange={e=>setCups(e.target.value)} placeholder=“1” style={inp} inputMode=“numeric”/></Fl>
</div>
<Fl l="メモ(任意)"><input value={memo} onChange={e=>setMemo(e.target.value)} placeholder=“一言メモ” style={inp}/></Fl>
{prev>0&&<div style={{background:C.accGlow,borderRadius:14,padding:16,textAlign:“center”,border:“1px solid rgba(255,140,26,0.18)”}}>
<div style={{fontSize:11,color:C.dim,marginBottom:4}}>スコア（プレビュー）</div>
<span style={{fontSize:28,fontWeight:900,color:C.acc}}>{prev.toLocaleString()}</span>
<span style={{fontSize:14,color:C.dim}}>pt = {fmtD(s2m(prev))}</span>
</div>}
<button onClick={submit} style={{…B,width:“100%”,padding:“16px”,background:`linear-gradient(135deg,${C.acc},${C.accDim})`,color:”#1a1a24”,fontSize:17,fontWeight:900,borderRadius:16}}>記録する</button>
</div>

  </div>;
}

// ============================================================
// RESULT
// ============================================================
function Result({sess,onBack}) {
const d=sess.totalDist||s2m(sess.totalScore||0);const rank=getRank(d);const[cp,setCp]=useState(false);
const dt=sess.destination?`\n📍 目的地: ${sess.destination.name}（${sess.arrived?"到着！":fmtD(Math.max((sess.destination.distance||0)-d,0))+" 残り"}）`:””;
const txt=`🏎️ 泥酔ドライブの記録\n\n📏 ${fmtD(d)} 走行${dt}\n🏅 ${rank.label}\n🍺 ${sess.totalCups||0}杯\n\n#泥酔ドライブ #DrunkDrive`;
const share=async()=>{if(navigator.share){try{await navigator.share({title:“泥酔ドライブ”,text:txt})}catch{}}else{try{await navigator.clipboard.writeText(txt);setCp(true);setTimeout(()=>setCp(false),2000)}catch{}}};

return <div style={{padding:“20px 18px 110px”}}>
<div style={{display:“flex”,alignItems:“center”,gap:10,marginBottom:20}}><SB onClick={onBack}>←</SB><h2 style={{fontSize:19,fontWeight:700,margin:0}}>🏁 結果</h2></div>
<div style={{background:`linear-gradient(135deg,#1e1a10,${C.card})`,borderRadius:22,padding:24,marginBottom:16,textAlign:“center”,border:“1px solid rgba(255,140,26,0.25)”,boxShadow:`0 8px 28px ${C.accGlow}`}}>
{sess.arrived&&<div style={{fontSize:40,marginBottom:4}}>🎉</div>}
<div style={{fontSize:42,fontWeight:900,color:C.acc}}>{fmtD(d)}</div>
<div style={{fontSize:13,color:C.dim,marginBottom:16}}>走行距離</div>
{sess.destination&&<div style={{fontSize:15,marginBottom:16,color:sess.arrived?C.success:C.text}}>{sess.destination.emoji||“📍”} {sess.destination.name} {sess.arrived?“到着！”:`まで残り ${fmtD(Math.max(sess.destination.distance-d,0))}`}</div>}
<div style={{background:“rgba(255,140,26,0.08)”,borderRadius:12,padding:14,border:“1px solid rgba(255,140,26,0.15)”}}><div style={{fontSize:18,fontWeight:900,color:C.acc}}>{rank.label}</div><div style={{fontSize:13,color:C.dim,marginTop:4}}>{rank.comment}</div></div>
</div>
<div style={{display:“flex”,gap:10,marginBottom:16}}><SC l=“杯数” v={`${sess.totalCups||0}杯`}/><SC l=“スコア” v={`${(sess.totalScore||0).toLocaleString()}pt`} a/></div>
{sess.drinks?.length>0&&<div style={{background:C.card,borderRadius:14,padding:14,marginBottom:16,border:`1px solid ${C.border}`}}><div style={{fontSize:12,color:C.dim,marginBottom:8}}>飲んだお酒</div>
{sess.drinks.map((dk,i)=><div key={dk.id||i} style={{display:“flex”,justifyContent:“space-between”,padding:“6px 0”,borderBottom:i<sess.drinks.length-1?`1px solid ${C.border}`:“none”,fontSize:14}}><span>{dk.name} ×{dk.cups}（{dk.volume}ml, {dk.abv}%）</span><span style={{color:C.acc}}>{dk.score.toLocaleString()}pt</span></div>)}
</div>}
<button onClick={share} style={{…B,width:“100%”,background:“linear-gradient(135deg,#4a9eff,#2d7dd2)”,color:”#fff”,fontSize:15,borderRadius:14}}>{cp?“✅ コピー済み”:“📤 共有する”}</button>

  </div>;
}

// ============================================================
// HISTORY
// ============================================================
function Hist({list,onBack,onView,onDel}) {
const[cf,setCf]=useState(null);
return <div style={{padding:“20px 18px 110px”}}>
<div style={{display:“flex”,alignItems:“center”,gap:10,marginBottom:20}}><SB onClick={onBack}>←</SB><h2 style={{fontSize:19,fontWeight:700,margin:0}}>📖 履歴</h2></div>
{!list.length?<div style={{textAlign:“center”,padding:40,color:C.dim}}><div style={{fontSize:44,marginBottom:12}}>🗺️</div><div style={{fontSize:15}}>まだ記録がありません</div></div>
:list.map(s=>{const dt=new Date(s.startedAt||s.endedAt);return<div key={s.id} style={{background:C.card,borderRadius:14,padding:14,marginBottom:10,border:`1px solid ${C.border}`}}>
<div style={{display:“flex”,justifyContent:“space-between”,marginBottom:8}}><span style={{fontSize:14,fontWeight:700}}>{dt.toLocaleDateString(“ja-JP”)}</span>{s.destination&&<span style={{fontSize:12,color:C.acc}}>{s.destination.name}{s.arrived?” ✅”:””}</span>}</div>
<div style={{display:“flex”,gap:14,fontSize:13,color:C.dim,marginBottom:10}}><span>{fmtD(s.totalDist)}</span><span>{s.totalScore?.toLocaleString()}pt</span><span>{s.totalCups}杯</span></div>
<div style={{display:“flex”,gap:8}}>
<button onClick={()=>onView(s)} style={{…B,flex:1,padding:“8px”,fontSize:13,background:C.accGlow,color:C.acc,border:“1px solid rgba(255,140,26,0.25)”,borderRadius:10}}>詳細</button>
{cf===s.id?<button onClick={()=>{onDel(s.id);setCf(null)}} style={{…B,padding:“8px 14px”,fontSize:13,background:“rgba(232,80,80,0.15)”,color:C.danger,border:“1px solid rgba(232,80,80,0.3)”,borderRadius:10}}>削除する</button>
:<button onClick={()=>setCf(s.id)} style={{…B,padding:“8px 14px”,fontSize:13,background:“transparent”,color:C.dim,border:`1px solid ${C.border}`,borderRadius:10}}>🗑️</button>}
</div></div>;})}

  </div>;
}

// ============================================================
// SETTINGS
// ============================================================
function Setts({val,set,onBack}) {
return <div style={{padding:“20px 18px 110px”}}>
<div style={{display:“flex”,alignItems:“center”,gap:10,marginBottom:20}}><SB onClick={onBack}>←</SB><h2 style={{fontSize:19,fontWeight:700,margin:0}}>⚙️ 設定</h2></div>
<div style={{background:C.card,borderRadius:14,padding:18,border:`1px solid ${C.border}`}}>
<div style={{display:“flex”,justifyContent:“space-between”,alignItems:“center”}}>
<div><div style={{fontSize:15,fontWeight:700}}>参考換算値の表示</div><div style={{fontSize:12,color:C.dim,marginTop:4}}>ルートマップで距離差分の参考値を表示</div></div>
<button onClick={()=>set({…val,showRef:!val.showRef})} style={{width:52,height:30,borderRadius:15,border:“none”,cursor:“pointer”,background:val.showRef?C.acc:C.border,position:“relative”,transition:“all 0.2s”}}><div style={{width:24,height:24,borderRadius:12,background:”#fff”,position:“absolute”,top:3,left:val.showRef?25:3,transition:“all 0.2s”}}/></button>
</div>
</div>
<div style={{marginTop:20,background:C.card,borderRadius:14,padding:18,border:`1px solid ${C.border}`}}>
<div style={{fontSize:14,fontWeight:700,marginBottom:10}}>このアプリについて</div>
<div style={{fontSize:12,color:C.dim,lineHeight:1.8}}>泥酔ドライブは飲酒量の記録とエンターテインメントを目的としたアプリです。飲酒を推奨するものではありません。表示される距離はエンタメ上の仮想値です。体調不良時・運転予定時は利用しないでください。飲酒運転は法律で禁止されています。</div>
</div>

  </div>;
}

// ============================================================
// SHARED COMPONENTS
// ============================================================
function SB({onClick,children}){return<button onClick={onClick} style={{…B,padding:“8px 13px”,background:C.card,color:C.text,border:`1px solid ${C.border}`,borderRadius:10,fontSize:14}}>{children}</button>}
function St({l,v,hi}){return<div style={{flex:1,textAlign:“center”}}><div style={{fontSize:10,color:C.dim,marginBottom:3}}>{l}</div><div style={{fontSize:hi?20:16,fontWeight:900,color:hi?C.acc:C.text}}>{v}</div></div>}
function SC({l,v,a}){return<div style={{flex:1,background:C.card,borderRadius:14,padding:14,textAlign:“center”,border:`1px solid ${C.border}`}}><div style={{fontSize:10,color:C.dim,marginBottom:4}}>{l}</div><div style={{fontSize:20,fontWeight:900,color:a?C.acc:C.text}}>{v}</div></div>}
function PBar({pct}){return<div style={{background:C.routeRem,borderRadius:8,height:10,overflow:“hidden”}}><div style={{width:`${Math.min(pct*100,100)}%`,height:“100%”,background:`linear-gradient(90deg,${C.acc},#ffaa44)`,borderRadius:8,transition:“width 0.8s ease”,boxShadow:`0 0 10px ${C.accGlow}`}}/></div>}
function Fl({l,e,children,f}){return<div style={{flex:f,display:“flex”,flexDirection:“column”,gap:5}}><label style={{fontSize:12,color:C.dim}}>{l}</label>{children}{e&&<span style={{fontSize:11,color:C.danger}}>{e}</span>}</div>}

// ============================================================
// OVERLAYS
// ============================================================
function WarnPop({onClose}){return<div style={{position:“fixed”,inset:0,background:C.overlay,display:“flex”,alignItems:“center”,justifyContent:“center”,zIndex:1000,padding:24}}>

  <div style={{background:C.card,borderRadius:22,padding:24,maxWidth:340,width:"100%",border:"1px solid rgba(232,80,80,0.25)",textAlign:"center"}}>
    <div style={{fontSize:44,marginBottom:10}}>⚠️</div><h3 style={{fontSize:19,color:C.danger,marginBottom:10}}>飲みすぎ注意</h3>
    <p style={{fontSize:13,lineHeight:1.7,color:C.dim,marginBottom:18}}>スコアが{WARNING_THRESHOLD.toLocaleString()}ptを超えました。体調に十分注意してください。無理な飲酒は絶対にやめましょう。</p>
    <button onClick={onClose} style={{...B,width:"100%",background:"rgba(232,80,80,0.12)",color:"#e8a0a0",border:"1px solid rgba(232,80,80,0.25)",borderRadius:14,fontSize:15}}>理解しました</button>
  </div></div>;}

function ArrFx({d}){return<div style={{position:“fixed”,inset:0,background:“rgba(0,0,0,0.85)”,display:“flex”,alignItems:“center”,justifyContent:“center”,zIndex:999,animation:“fio 3.5s ease forwards”}}>

  <div style={{textAlign:"center",animation:"si 0.5s ease"}}><div style={{fontSize:72}}>{d?.emoji||"🎉"}</div><div style={{fontFamily:fontD,fontSize:30,color:C.acc,marginTop:12}}>{d?.name||"目的地"}</div><div style={{fontSize:18,color:C.text,marginTop:8}}>に到着！</div></div>
  <style>{`@keyframes fio{0%{opacity:0}10%{opacity:1}80%{opacity:1}100%{opacity:0}}@keyframes si{0%{transform:scale(.5);opacity:0}100%{transform:scale(1);opacity:1}}`}</style></div>;}

function SafetyBar(){return<div style={{position:“fixed”,bottom:0,left:0,right:0,background:“linear-gradient(transparent,rgba(12,12,18,0.95) 30%)”,padding:“28px 18px 10px”,textAlign:“center”,zIndex:100,pointerEvents:“none”}}>

  <div style={{fontSize:9,color:"rgba(122,118,112,0.55)",lineHeight:1.5,maxWidth:480,margin:"0 auto"}}>⚠️ 飲酒運転は法律で禁止 · 飲みすぎ注意 · 体調不良時は利用中止 · エンタメ目的の仮想距離表示です · 健康指標ではありません</div></div>;}
