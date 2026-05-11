# HVAC-Distribution-Monthly-Defrost
HVAC Distribution News
import { useState, useEffect } from "react";

const SECTION_COLORS = {
  news:      { border:"#3B82F6", badge:"#DBEAFE", badgeText:"#1D4ED8" },
  stats:     { border:"#22C55E", badge:"#DCFCE7", badgeText:"#15803D" },
  pe:        { border:"#8B5CF6", badge:"#EDE9FE", badgeText:"#6D28D9" },
  workforce: { border:"#F97316", badge:"#FFEDD5", badgeText:"#C2410C" }
};

const COMPASS_HELPS = [
  { painPoint:"Market Visibility Gaps", insight:"Distributors lack real-time data on what's moving, what's stalling, and where margin is leaking.", compassRole:"Compass surfaces account-level intelligence so your team always knows where to focus — before revenue walks out the door." },
  { painPoint:"PE-Backed Competitor Pressure", insight:"Consolidated rivals have deeper pockets, better tech, and faster fulfillment cycles.", compassRole:"Compass gives independent distributors enterprise-grade sales intelligence without the enterprise overhead." },
  { painPoint:"Technician Shortage Ripple Effects", insight:"Fewer techs mean slower installs, longer sales cycles, and contractors who need more hand-holding.", compassRole:"Compass helps your reps understand contractor pain at a deeper level — turning consultative selling into a competitive edge." },
  { painPoint:"A2L Transition Complexity", insight:"R-410A install bans and new refrigerant requirements are creating confusion and hesitation across the channel.", compassRole:"Compass equips your team with the talking points and customer-specific context to guide contractors through the transition confidently." }
];

const SECTIONS = [
  {
    id:"news", title:"Industry News & Trends", icon:"📰", subtitle:"What's moving the needle right now",
    prompt:`Search the web for the 3-4 most important HVAC distribution industry news and trends from 2025-2026. Return ONLY a raw JSON array — no preamble, no backticks, no markdown. Each object must have exactly: headline (string), summary (2 sentences max, string), source (publication name, string), whyItMatters (1 sentence impact on HVAC distributors, string).`
  },
  {
    id:"stats", title:"Key Market Stats", icon:"📊", subtitle:"Numbers that drive decisions",
    prompt:`Search the web for 4-5 current HVAC distribution market statistics and data points for 2025-2026 (market size, growth, pricing, channel trends). Return ONLY a raw JSON array — no preamble, no backticks, no markdown. Each object must have exactly: stat (specific number or %, string), metric (what it measures, 5-8 words, string), source (string), implication (1 sentence for distributors, string).`
  },
  {
    id:"pe", title:"PE Consolidation & Big Moves", icon:"🏢", subtitle:"Who's buying, merging, and disrupting",
    prompt:`Search the web for 3-4 recent private equity deals, M&A activity, or major strategic moves in HVAC distribution in 2025-2026. Return ONLY a raw JSON array — no preamble, no backticks, no markdown. Each object must have exactly: companies (string), dealType (e.g. Acquisition, PE Investment, Merger, string), description (2-3 sentences, string), impactOnIndependents (1-2 sentences for independent distributors, string).`
  },
  {
    id:"workforce", title:"Technician Shortage Insights", icon:"👷", subtitle:"The workforce gap shaping the whole channel",
    prompt:`Search the web for 3-4 current findings on the HVAC technician shortage and workforce challenges in 2025-2026. Return ONLY a raw JSON array — no preamble, no backticks, no markdown. Each object must have exactly: finding (4-7 word title, string), dataPoint (key number or fact, string), context (2 sentences, string), implication (1 sentence impact on HVAC distributors, string).`
  }
];

const Skeleton = () => (
  <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(260px,1fr))",gap:12}}>
    {[1,2,3].map(i=>(
      <div key={i} style={{background:"#fff",borderRadius:12,padding:20,border:"1px solid #E5E0D8"}}>
        {[70,100,85].map((w,j)=>(
          <div key={j} style={{height:j===0?15:11,background:"#F0EDE8",borderRadius:6,width:`${w}%`,marginBottom:8,animation:"pulse 1.5s ease-in-out infinite",animationDelay:`${j*0.1}s`}}/>
        ))}
      </div>
    ))}
  </div>
);

const ErrorBox = ({onRetry}) => (
  <div style={{background:"#FEF2F2",border:"1px solid #FECACA",borderRadius:12,padding:20,textAlign:"center",color:"#DC2626",fontSize:13}}>
    Could not load data.{" "}
    <button onClick={onRetry} style={{color:"#DC2626",fontWeight:700,background:"none",border:"none",cursor:"pointer",textDecoration:"underline"}}>Try again</button>
  </div>
);

const NewsCard = ({item,c}) => (
  <div style={{background:"#fff",borderRadius:12,padding:18,border:`1px solid ${c.border}`,borderLeft:`4px solid ${c.border}`,marginBottom:10}}>
    <div style={{fontSize:13,fontWeight:700,color:"#1B3A6B",marginBottom:6,lineHeight:1.35}}>{item.headline}</div>
    <div style={{fontSize:12,color:"#4B5563",marginBottom:10,lineHeight:1.55}}>{item.summary}</div>
    <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",gap:8,flexWrap:"wrap"}}>
      <div style={{fontSize:11,background:c.badge,color:c.badgeText,padding:"3px 10px",borderRadius:20,fontWeight:600,lineHeight:1.4}}>📌 {item.whyItMatters}</div>
      <div style={{fontSize:10,color:"#9CA3AF",paddingTop:4,whiteSpace:"nowrap"}}>via {item.source}</div>
    </div>
  </div>
);

const StatCard = ({item,c}) => (
  <div style={{background:"#fff",borderRadius:12,padding:20,border:`1px solid ${c.border}`,borderTop:`4px solid ${c.border}`,textAlign:"center"}}>
    <div style={{fontSize:30,fontWeight:900,color:"#1B3A6B",marginBottom:4}}>{item.stat}</div>
    <div style={{fontSize:12,fontWeight:600,color:"#374151",marginBottom:6}}>{item.metric}</div>
    <div style={{fontSize:11,color:"#6B7280",marginBottom:8,lineHeight:1.45}}>{item.implication}</div>
    <div style={{fontSize:10,color:"#9CA3AF"}}>Source: {item.source}</div>
  </div>
);

const PECard = ({item,c}) => (
  <div style={{background:"#fff",borderRadius:12,padding:18,border:`1px solid ${c.border}`,borderLeft:`4px solid ${c.border}`,marginBottom:10}}>
    <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:8,flexWrap:"wrap"}}>
      <span style={{fontSize:11,background:c.badge,color:c.badgeText,padding:"2px 10px",borderRadius:20,fontWeight:700}}>{item.dealType}</span>
      <span style={{fontSize:13,fontWeight:700,color:"#1B3A6B"}}>{item.companies}</span>
    </div>
    <div style={{fontSize:12,color:"#4B5563",marginBottom:10,lineHeight:1.55}}>{item.description}</div>
    <div style={{fontSize:11,background:"#FFFBEB",color:"#92400E",padding:"8px 12px",borderRadius:8,lineHeight:1.45}}>
      <span style={{fontWeight:700}}>For independents: </span>{item.impactOnIndependents}
    </div>
  </div>
);

const WorkforceCard = ({item,c}) => (
  <div style={{background:"#fff",borderRadius:12,padding:18,border:`1px solid ${c.border}`,borderLeft:`4px solid ${c.border}`,marginBottom:10}}>
    <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",gap:8,marginBottom:8}}>
      <div style={{fontSize:13,fontWeight:700,color:"#1B3A6B"}}>{item.finding}</div>
      <div style={{fontSize:13,fontWeight:800,color:c.badgeText,background:c.badge,padding:"2px 10px",borderRadius:20,whiteSpace:"nowrap"}}>{item.dataPoint}</div>
    </div>
    <div style={{fontSize:12,color:"#4B5563",marginBottom:8,lineHeight:1.55}}>{item.context}</div>
    <div style={{fontSize:11,color:"#D97706",fontWeight:600}}>↳ {item.implication}</div>
  </div>
);

export default function App() {
  const [data, setData] = useState({});
  const [loading, setLoading] = useState({});
  const [errors, setErrors] = useState({});
  const [lastRefreshed, setLastRefreshed] = useState(null);

  const fetchSection = async (s) => {
    setLoading(p=>({...p,[s.id]:true}));
    setErrors(p=>({...p,[s.id]:false}));
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages",{
        method:"POST",
        headers:{"Content-Type":"application/json"},
        body:JSON.stringify({
          model:"claude-sonnet-4-20250514",
          max_tokens:1000,
          tools:[{type:"web_search_20250305",name:"web_search"}],
          system:"You are an HVAC industry analyst. Always respond with ONLY a valid raw JSON array — no preamble, no markdown, no code fences, no explanation. Just the JSON array.",
          messages:[{role:"user",content:s.prompt}]
        })
      });
      const result = await res.json();
      const txt = (result.content||[]).filter(b=>b.type==="text").map(b=>b.text).join("").trim();
      const clean = txt.replace(/^```(?:json)?\s*/i,"").replace(/\s*```$/i,"").trim();
      const parsed = JSON.parse(clean);
      setData(p=>({...p,[s.id]:parsed}));
    } catch {
      setErrors(p=>({...p,[s.id]:true}));
    }
    setLoading(p=>({...p,[s.id]:false}));
  };

  const fetchAll = () => {
    setLastRefreshed(new Date());
    SECTIONS.forEach(fetchSection);
  };

  useEffect(()=>{fetchAll();},[]);

  const fmtTime = d => d ? d.toLocaleTimeString([],{hour:"2-digit",minute:"2-digit"}) : "";

  const renderItems = (s) => {
    const c = SECTION_COLORS[s.id];
    const items = data[s.id];
    if (!items) return null;
    if (s.id==="stats") return (
      <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(190px,1fr))",gap:12}}>
        {items.map((item,i)=><StatCard key={i} item={item} c={c}/>)}
      </div>
    );
    return items.map((item,i) =>
      s.id==="news" ? <NewsCard key={i} item={item} c={c}/> :
      s.id==="pe"   ? <PECard   key={i} item={item} c={c}/> :
                      <WorkforceCard key={i} item={item} c={c}/>
    );
  };

  return (
    <div style={{fontFamily:"'Inter',system-ui,sans-serif",background:"#F8F4EE",minHeight:"100vh"}}>
      {/* Hero */}
      <div style={{background:"linear-gradient(135deg,#1B3A6B 0%,#0F2347 100%)",padding:"40px 24px 32px",textAlign:"center",position:"relative",overflow:"hidden"}}>
        <div style={{position:"absolute",inset:0,backgroundImage:"radial-gradient(circle at 20% 80%,rgba(232,168,56,.15) 0%,transparent 50%),radial-gradient(circle at 80% 20%,rgba(59,130,246,.1) 0%,transparent 50%)"}}/>
        <div style={{position:"relative"}}>
          <div style={{display:"inline-flex",alignItems:"center",gap:10,background:"rgba(255,255,255,.12)",padding:"6px 18px",borderRadius:24,marginBottom:16,backdropFilter:"blur(8px)"}}>
            <span style={{color:"#E8A838",fontSize:15,fontWeight:800,letterSpacing:".5px"}}>🧭 COMPASS</span>
            <span style={{color:"rgba(255,255,255,.5)",fontSize:12}}>by EBM / BlueOps</span>
          </div>
          <h1 style={{color:"#fff",fontSize:26,fontWeight:900,margin:"0 0 8px",letterSpacing:"-.5px"}}>HVAC Distribution Intelligence Hub</h1>
          <p style={{color:"rgba(255,255,255,.72)",fontSize:13,margin:"0 0 22px",maxWidth:460,marginLeft:"auto",marginRight:"auto",lineHeight:1.65}}>Your industry, decoded. Live insights on the trends, moves, and forces shaping HVAC distribution — curated for the people running the channel.</p>
          <div style={{display:"flex",alignItems:"center",justifyContent:"center",gap:16,flexWrap:"wrap"}}>
            <span style={{fontSize:12,color:"rgba(255,255,255,.45)"}}>
              {lastRefreshed ? `Last updated: ${fmtTime(lastRefreshed)}` : "Loading live data..."}
            </span>
            <button onClick={fetchAll} style={{background:"#E8A838",color:"#1B3A6B",border:"none",padding:"8px 22px",borderRadius:24,fontSize:13,fontWeight:700,cursor:"pointer"}}>↻ Refresh All</button>
          </div>
        </div>
      </div>

      {/* Sections */}
      <div style={{maxWidth:900,margin:"0 auto",padding:"32px 20px"}}>
        {SECTIONS.map(s => {
          const c = SECTION_COLORS[s.id];
          const isLoading = loading[s.id];
          const hasError = errors[s.id];
          return (
            <div key={s.id} style={{marginBottom:44}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:14,flexWrap:"wrap",gap:8}}>
                <div>
                  <h2 style={{margin:0,fontSize:19,fontWeight:800,color:"#1B3A6B"}}>{s.icon} {s.title}</h2>
                  <p style={{margin:"3px 0 0",fontSize:12,color:"#6B7280"}}>{s.subtitle}</p>
                </div>
                <button onClick={()=>fetchSection(s)} disabled={isLoading} style={{background:"transparent",border:`1px solid ${c.border}`,color:c.badgeText,padding:"5px 14px",borderRadius:20,fontSize:12,fontWeight:600,cursor:isLoading?"not-allowed":"pointer",opacity:isLoading?.5:1}}>
                  {isLoading?"Refreshing...":"↻ Refresh"}
                </button>
              </div>
              {isLoading && <Skeleton/>}
              {hasError && !isLoading && <ErrorBox onRetry={()=>fetchSection(s)}/>}
              {!isLoading && !hasError && renderItems(s)}
            </div>
          );
        })}

        {/* How Compass Helps */}
        <div style={{marginBottom:44}}>
          <h2 style={{margin:"0 0 4px",fontSize:19,fontWeight:800,color:"#1B3A6B"}}>🧭 How Compass Helps</h2>
          <p style={{margin:"0 0 14px",fontSize:12,color:"#6B7280"}}>Connecting industry pain to practical solutions</p>
          <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(260px,1fr))",gap:12}}>
            {COMPASS_HELPS.map((item,i)=>(
              <div key={i} style={{background:"#fff",borderRadius:12,padding:20,border:"1px solid #E5E0D8",borderTop:"4px solid #E8A838"}}>
                <div style={{fontSize:10,fontWeight:700,color:"#6B7280",textTransform:"uppercase",letterSpacing:".6px",marginBottom:6}}>{item.painPoint}</div>
                <div style={{fontSize:12,color:"#4B5563",marginBottom:10,lineHeight:1.55,fontStyle:"italic"}}>"{item.insight}"</div>
                <div style={{fontSize:12,color:"#1B3A6B",fontWeight:600,lineHeight:1.55}}>→ {item.compassRole}</div>
              </div>
            ))}
          </div>
        </div>

        {/* Footer CTA */}
        <div style={{background:"linear-gradient(135deg,#1B3A6B,#0F2347)",borderRadius:16,padding:"36px 24px",textAlign:"center"}}>
          <h3 style={{color:"#fff",margin:"0 0 8px",fontSize:18,fontWeight:800}}>Ready to put this intelligence to work?</h3>
          <p style={{color:"rgba(255,255,255,.7)",fontSize:13,margin:"0 0 22px",maxWidth:420,marginLeft:"auto",marginRight:"auto",lineHeight:1.6}}>Compass by EBM/BlueOps turns market insight into sales motion — built for HVAC distributors who want to win the channel.</p>
          <button style={{background:"#E8A838",color:"#1B3A6B",border:"none",padding:"12px 30px",borderRadius:24,fontSize:14,fontWeight:700,cursor:"pointer"}}>Connect with the Compass Team →</button>
        </div>
      </div>

      <style>{`@keyframes pulse{0%,100%{opacity:1}50%{opacity:.4}}`}</style>
    </div>
  );
}
