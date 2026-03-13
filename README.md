import { useState, useRef, useEffect } from "react";

const JOURS_LONGS  = ["Lundi","Mardi","Mercredi","Jeudi","Vendredi","Samedi","Dimanche"];
const JOURS_COURTS = ["Lun","Mar","Mer","Jeu","Ven","Sam","Dim"];
const MOIS = ["Janvier","Février","Mars","Avril","Mai","Juin","Juillet","Août","Septembre","Octobre","Novembre","Décembre"];
const HEURES = Array.from({length:13},(_,i)=>i+7);

const CATEGORIES = [
  {id:"travail",  label:"Travail",    couleur:"#6366f1"},
  {id:"perso",    label:"Personnel",  couleur:"#10b981"},
  {id:"sante",    label:"Santé",      couleur:"#f43f5e"},
  {id:"urgent",   label:"Urgent",     couleur:"#f97316"},
  {id:"objectif", label:"Objectif",   couleur:"#f59e0b"},
];
const PRI_C = {haute:"#f43f5e",moyenne:"#f59e0b",basse:"#10b981"};
const PRI_I = {haute:"▲",moyenne:"●",basse:"▼"};

const DOMAINES = [
  {id:"carriere",  label:"Carrière",     ic:"💼", couleur:"#6366f1"},
  {id:"sante",     label:"Santé",        ic:"❤️", couleur:"#f43f5e"},
  {id:"finances",  label:"Finances",     ic:"💰", couleur:"#f59e0b"},
  {id:"relations", label:"Relations",    ic:"👥", couleur:"#10b981"},
  {id:"perso",     label:"Croissance",   ic:"🌱", couleur:"#8b5cf6"},
  {id:"loisirs",   label:"Loisirs",      ic:"🎯", couleur:"#ec4899"},
];

const NAV = [
  {id:"ia",       ic:"✦", label:"IA"},
  {id:"jour",     ic:"◈", label:"Jour"},
  {id:"semaine",  ic:"▦", label:"Semaine"},
  {id:"objectifs",ic:"◎", label:"Objectifs"},
  {id:"taches",   ic:"◉", label:"Tâches"},
];

const HAB_INIT = [
  {id:1,nom:"Méditation",ic:"🧘",c:"#8b5cf6"},
  {id:2,nom:"Sport",     ic:"🏃",c:"#10b981"},
  {id:3,nom:"Lecture",   ic:"📚",c:"#f59e0b"},
  {id:4,nom:"Eau 2L",    ic:"💧",c:"#3b82f6"},
];

const OBJECTIFS_INIT = [
  {id:1, titre:"Lancer mon projet freelance", domaine:"carriere", echeance:"2025-06-30", progres:35, priorite:"haute", etapes:[
    {id:1,titre:"Créer mon portfolio",fait:true},
    {id:2,titre:"Trouver 3 premiers clients",fait:false},
    {id:3,titre:"Fixer mes tarifs",fait:false},
  ]},
  {id:2, titre:"Courir un 10km", domaine:"sante", echeance:"2025-05-15", progres:60, priorite:"moyenne", etapes:[
    {id:1,titre:"Courir 3x par semaine",fait:true},
    {id:2,titre:"Atteindre 7km sans arrêt",fait:true},
    {id:3,titre:"S'inscrire à une course",fait:false},
  ]},
  {id:3, titre:"Épargner 5000€", domaine:"finances", echeance:"2025-12-31", progres:20, priorite:"haute", etapes:[
    {id:1,titre:"Budget mensuel détaillé",fait:true},
    {id:2,titre:"Automatiser épargne 300€/mois",fait:false},
    {id:3,titre:"Réduire abonnements inutiles",fait:false},
  ]},
];

const SUGGESTIONS_IA = [
  "🎯 Aide-moi à définir mes objectifs de vie",
  "📅 Planifie ma semaine selon mes objectifs",
  "🔍 Recherche des conseils pour ma carrière",
  "💡 Comment améliorer ma productivité ?",
  "📊 Analyse mes objectifs et donne-moi un plan",
  "🌱 Crée un plan de développement personnel",
];

function semaineDe(date){
  const d=new Date(date),j=d.getDay(),diff=d.getDate()-j+(j===0?-6:1);
  const lun=new Date(d.setDate(diff));
  return Array.from({length:7},(_,i)=>{const dd=new Date(lun);dd.setDate(lun.getDate()+i);return dd;});
}
function memeJour(a,b){return a.toDateString()===b.toDateString();}
function catDe(id){return CATEGORIES.find(c=>c.id===id)||CATEGORIES[0];}
function domaineDe(id){return DOMAINES.find(d=>d.id===id)||DOMAINES[0];}

export default function App(){
  const TODAY = new Date();
  const [vue,setVue]           = useState("ia");
  const [sel,setSel]           = useState(new Date());
  const [taches,setTaches]     = useState(()=>{
    const d=TODAY.toDateString();
    return [
      {id:1,titre:"Réunion d'équipe",    date:d,h:"09:00",dur:60,cat:"travail",  pri:"haute",  fait:false,notes:""},
      {id:2,titre:"Séance de sport",     date:d,h:"07:00",dur:45,cat:"sante",    pri:"moyenne",fait:true, notes:""},
      {id:3,titre:"Travailler portfolio",date:d,h:"14:00",dur:90,cat:"objectif", pri:"haute",  fait:false,notes:""},
    ];
  });
  const [habs,setHabs]         = useState(HAB_INIT);
  const [jrnl,setJrnl]         = useState({});
  const [objectifs,setObjectifs]= useState(OBJECTIFS_INIT);
  const [modal,setModal]       = useState(false);
  const [modalObj,setModalObj] = useState(false);
  const [editT,setEditT]       = useState(null);
  const [selObj,setSelObj]     = useState(null);
  const [form,setForm]         = useState({titre:"",h:"09:00",dur:30,cat:"travail",pri:"moyenne",notes:""});
  const [formObj,setFormObj]   = useState({titre:"",domaine:"carriere",echeance:"",priorite:"haute"});

  // ── IA ───────────────────────────────────────────────────
  const [msgs,setMsgs]     = useState([{
    role:"assistant",
    content:"Bonjour ! ✦ Je suis votre **assistant de vie Planify**.\n\nJe peux vous aider à :\n\n🎯 **Définir & suivre vos objectifs de vie**\n📅 **Planifier votre semaine intelligemment**\n🔍 **Faire des recherches fiables** sur le web\n🗓️ **Synchroniser Google Calendar**\n💡 **Conseils personnalisés** selon votre profil\n\nCommencez par me parler de vous et de vos ambitions !"
  }]);
  const [input,setInput]   = useState("");
  const [loading,setLoading]= useState(false);
  const endRef = useRef(null);
  useEffect(()=>{ endRef.current?.scrollIntoView({behavior:"smooth"}); },[msgs,loading]);

  const selStr   = sel.toDateString();
  const todayStr = TODAY.toDateString();
  const tJour    = taches.filter(t=>t.date===selStr).sort((a,b)=>a.h.localeCompare(b.h));
  const datesSem = semaineDe(sel);
  const fC       = tJour.filter(t=>t.fait).length;
  const habFait  = habs.filter(h=>!!jrnl[`${todayStr}-${h.id}`]).length;

  const toggle   = id => setTaches(p=>p.map(t=>t.id===id?{...t,fait:!t.fait}:t));
  const del      = id => setTaches(p=>p.filter(t=>t.id!==id));
  const togH     = id => { const k=`${todayStr}-${id}`; setJrnl(p=>({...p,[k]:!p[k]})); };
  const togEtape = (oid,eid) => setObjectifs(p=>p.map(o=>o.id===oid?{...o,etapes:o.etapes.map(e=>e.id===eid?{...e,fait:!e.fait}:e)}:o));

  function ouvrir(t=null){
    setEditT(t);
    setForm(t?{titre:t.titre,h:t.h,dur:t.dur,cat:t.cat,pri:t.pri,notes:t.notes}:{titre:"",h:"09:00",dur:30,cat:"travail",pri:"moyenne",notes:""});
    setModal(true);
  }
  function sauver(){
    if(!form.titre.trim()) return;
    if(editT) setTaches(p=>p.map(t=>t.id===editT.id?{...t,...form}:t));
    else      setTaches(p=>[...p,{...form,id:Date.now(),date:selStr,fait:false}]);
    setModal(false);
  }
  function ajouterObj(){
    if(!formObj.titre.trim()) return;
    setObjectifs(p=>[...p,{...formObj,id:Date.now(),progres:0,etapes:[]}]);
    setModalObj(false);
    setFormObj({titre:"",domaine:"carriere",echeance:"",priorite:"haute"});
  }

  // ── APPEL IA ─────────────────────────────────────────────
  async function envoyerIA(texte){
    const userMsg=(texte||input).trim();
    if(!userMsg||loading) return;
    setInput("");
    setMsgs(p=>[...p,{role:"user",content:userMsg}]);
    setLoading(true);

    const aujourd = TODAY.toDateString();
    const semCtx  = datesSem.map((d,i)=>({
      jour:JOURS_LONGS[i], date:d.toDateString(),
      taches:taches.filter(t=>t.date===d.toDateString()).map(t=>({titre:t.titre,heure:t.h,cat:t.cat,pri:t.pri,fait:t.fait}))
    }));
    const objCtx = objectifs.map(o=>({
      titre:o.titre, domaine:o.domaine, progres:`${o.progres}%`,
      echeance:o.echeance, priorite:o.priorite,
      etapes:o.etapes.map(e=>({titre:e.titre,fait:e.fait}))
    }));

    const systemPrompt=`Tu es Planify IA, un coach de vie et assistant planificateur qui parle UNIQUEMENT en français. Tu es bienveillant, motivant et très pratique.

PROFIL UTILISATEUR & AGENDA:
- Date: ${JOURS_LONGS[(TODAY.getDay()+6)%7]} ${TODAY.getDate()} ${MOIS[TODAY.getMonth()]} ${TODAY.getFullYear()}
- Tâches aujourd'hui: ${JSON.stringify(taches.filter(t=>t.date===aujourd).map(t=>({titre:t.titre,heure:t.h,cat:t.cat,fait:t.fait})))}
- Planning semaine: ${JSON.stringify(semCtx)}
- Objectifs de vie: ${JSON.stringify(objCtx)}
- Habitudes suivies: ${habs.map(h=>h.nom).join(", ")}

TES CAPACITÉS:
1. AJOUTER TÂCHES — inclus ce bloc dans ta réponse si tu veux créer des tâches:
   [TACHES:{"taches":[{"titre":"Nom","date":"${aujourd}","h":"10:00","dur":30,"cat":"travail","pri":"moyenne","notes":""}]}]
   Dates disponibles: ${datesSem.map((d,i)=>`${JOURS_LONGS[i]}="${d.toDateString()}"`).join(", ")}
   Catégories: travail, perso, sante, urgent, objectif — Priorités: haute, moyenne, basse

2. RECHERCHE WEB — tu as accès à la recherche web en temps réel pour donner des conseils fiables et actualisés

3. COACHING DE VIE — aide à définir des objectifs SMART, stratégies de productivité, développement personnel

4. ANALYSE AGENDA — identifie les surcharges, temps libres, aligne les tâches avec les objectifs

STYLE: Sois concis mais complet. Utilise des emojis avec parcimonie. Structure tes réponses avec des listes quand c'est utile. Sois motivant et concret.`;

    try{
      const history = msgs.slice(-8).map(m=>({role:m.role,content:m.content}));
      const res = await fetch("https://api.anthropic.com/v1/messages",{
        method:"POST",
        headers:{"Content-Type":"application/json"},
        body:JSON.stringify({
          model:"claude-sonnet-4-20250514",
          max_tokens:1200,
          system:systemPrompt,
          tools:[{type:"web_search_20250305",name:"web_search"}],
          messages:[...history,{role:"user",content:userMsg}]
        })
      });
      const data = await res.json();

      // Extraire texte et sources
      let texte="", srcs=[];
      if(data.content){
        for(const blk of data.content){
          if(blk.type==="text") texte+=blk.text;
          if(blk.type==="tool_result"){
            try{
              const r=JSON.parse(blk.content||"{}");
              if(r.results) srcs=[...srcs,...r.results.slice(0,3).map(x=>({titre:x.title,url:x.url}))];
            }catch(e){}
          }
        }
      }
      if(!texte) texte="Désolé, je n'ai pas pu répondre.";

      // Parser tâches
      const match=texte.match(/\[TACHES:\s*({[\s\S]*?})\]/);
      if(match){
        try{
          const parsed=JSON.parse(match[1]);
          if(parsed.taches?.length){
            setTaches(p=>[...p,...parsed.taches.map(t=>({...t,id:Date.now()+Math.random(),fait:false,dur:t.dur||30,cat:t.cat||"travail",pri:t.pri||"moyenne",notes:t.notes||""}))]);
          }
        }catch(e){}
      }

      const propre=texte.replace(/\[TACHES:[\s\S]*?\]/g,"").trim();
      setMsgs(p=>[...p,{role:"assistant",content:propre,sources:srcs}]);
    }catch(e){
      setMsgs(p=>[...p,{role:"assistant",content:"❌ Erreur de connexion. Vérifiez votre réseau."}]);
    }
    setLoading(false);
  }

  // ── CSS ──────────────────────────────────────────────────
  const css=`
    @import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@400;600;700&family=Outfit:wght@300;400;500;600&display=swap');
    *{margin:0;padding:0;box-sizing:border-box;}
    html,body,#root{height:100%;background:#07090e;overflow:hidden;}
    ::-webkit-scrollbar{width:3px;}::-webkit-scrollbar-track{background:transparent;}::-webkit-scrollbar-thumb{background:#1a1f2e;border-radius:2px;}

    .app{display:flex;flex-direction:column;height:100vh;background:#07090e;font-family:'Outfit',sans-serif;color:#ccd0e0;max-width:480px;margin:0 auto;}

    /* HEADER */
    .hdr{padding:13px 18px 10px;border-bottom:1px solid #0d1018;display:flex;align-items:center;justify-content:space-between;flex-shrink:0;}
    .logo{font-family:'Cormorant Garamond',serif;font-size:23px;font-weight:700;color:#fff;letter-spacing:-.5px;}
    .logo-ia{font-size:13px;color:#6366f1;margin-left:4px;}
    .logo-sub{font-size:10px;color:#1e2435;margin-top:1px;}
    .btn-sm{padding:6px 14px;border-radius:8px;border:1px solid #0d1018;background:#0b0e16;color:#4a5270;font-family:'Outfit',sans-serif;font-size:12px;font-weight:500;cursor:pointer;transition:all .15s;}
    .btn-sm:hover{border-color:#6366f1;color:#6366f1;}
    .btn-sm.pri{background:#6366f1;border-color:#6366f1;color:#fff;}
    .btn-sm.pri:hover{background:#5153cc;}

    /* NAV DATE */
    .nav-date{display:flex;align-items:center;justify-content:space-between;padding:9px 18px;border-bottom:1px solid #0d1018;flex-shrink:0;}
    .nd-btn{width:28px;height:28px;border:1px solid #0d1018;background:#0b0e16;color:#6366f1;border-radius:7px;cursor:pointer;font-size:16px;display:flex;align-items:center;justify-content:center;}
    .nd-txt{font-size:12px;color:#4a5270;font-weight:500;text-align:center;}
    .auj-b{font-size:9px;background:#6366f115;color:#6366f1;padding:2px 7px;border-radius:20px;font-weight:600;letter-spacing:.5px;display:block;text-align:center;margin-top:2px;}

    /* SCROLL */
    .scroll{flex:1;overflow-y:auto;padding:14px 16px 8px;}

    /* STATS */
    .stats{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:14px;}
    .stat{background:#0b0e16;border:1px solid #0d1018;border-radius:12px;padding:12px 10px;}
    .sv{font-family:'Cormorant Garamond',serif;font-size:26px;color:#fff;line-height:1;}
    .sl{font-size:9px;color:#1e2435;margin-top:2px;text-transform:uppercase;letter-spacing:.5px;}
    .sb{height:2px;background:#0d1018;border-radius:1px;margin-top:8px;overflow:hidden;}
    .sf{height:100%;border-radius:1px;transition:width .5s;}

    /* CARTE */
    .carte{background:#0b0e16;border:1px solid #0d1018;border-radius:14px;padding:16px;margin-bottom:14px;}
    .ct{font-size:9px;font-weight:600;letter-spacing:2px;text-transform:uppercase;color:#1e2435;margin-bottom:14px;}
    .ch{display:flex;align-items:center;justify-content:space-between;margin-bottom:14px;}

    /* TIMELINE */
    .tl{display:flex;gap:8px;min-height:46px;align-items:flex-start;}
    .tlh{font-size:10px;color:#1a1f2e;width:26px;flex-shrink:0;padding-top:4px;}
    .tlf{width:1px;background:#0d1018;flex-shrink:0;margin-top:10px;position:relative;}
    .tlp{width:5px;height:5px;background:#1a1f2e;border-radius:50%;position:absolute;top:-2.5px;left:-2px;}
    .tls{flex:1;padding-bottom:4px;}

    /* TÂCHE */
    .tache{display:flex;align-items:center;gap:8px;background:#0f1220;border-radius:9px;padding:9px 10px;border-left:2.5px solid #6366f1;margin-bottom:5px;cursor:pointer;transition:all .15s;}
    .tache:active{transform:scale(.99);}
    .tache.fait{opacity:.35;}
    .tache.fait .tt{text-decoration:line-through;}
    .chk{width:17px;height:17px;border-radius:5px;border:1.5px solid #1a1f2e;display:flex;align-items:center;justify-content:center;font-size:9px;cursor:pointer;transition:all .15s;flex-shrink:0;}
    .chk.on{background:#6366f1;border-color:#6366f1;color:#fff;}
    .tt{font-size:13px;font-weight:500;color:#b8bdd0;flex:1;}
    .tm{font-size:10px;color:#1e2435;}
    .acts{display:flex;gap:3px;}
    .ibtn{width:22px;height:22px;border:none;background:#1a1f2e;border-radius:5px;cursor:pointer;color:#3a4060;font-size:10px;display:flex;align-items:center;justify-content:center;}
    .ibtn:hover{color:#ccd0e0;}
    .add-btn{width:100%;padding:9px;border:1.5px dashed #0d1018;border-radius:9px;background:transparent;color:#1e2435;font-family:'Outfit',sans-serif;font-size:12px;cursor:pointer;margin-top:6px;transition:all .15s;}
    .add-btn:hover{border-color:#6366f1;color:#6366f1;}

    /* SEMAINE */
    .sem{display:grid;grid-template-columns:repeat(7,1fr);gap:4px;margin-bottom:14px;}
    .sd{background:#0f1220;border:1.5px solid transparent;border-radius:9px;padding:7px 3px;text-align:center;cursor:pointer;transition:all .15s;}
    .sd.auj{border-color:#2a3050;}
    .sd.sel{border-color:#6366f1;background:#6366f110;}
    .sdn{font-size:9px;color:#2a3045;font-weight:600;}
    .sdd{font-family:'Cormorant Garamond',serif;font-size:17px;color:#fff;margin:1px 0;}
    .sd.sel .sdd{color:#6366f1;}
    .sdpt{width:4px;height:4px;border-radius:50%;background:#6366f1;margin:0 auto;}
    .sdc{font-size:8px;color:#1e2435;margin-top:2px;}
    .sgt{font-size:11px;font-weight:600;color:#4a5270;margin-bottom:6px;}
    .stache{display:flex;gap:7px;align-items:flex-start;padding:7px 9px;background:#0f1220;border-radius:8px;margin-bottom:4px;}
    .sdot{width:6px;height:6px;border-radius:50%;flex-shrink:0;margin-top:3px;}
    .stt{font-size:12px;font-weight:500;color:#8b91a8;}
    .ssb{font-size:10px;color:#1e2435;margin-top:1px;}

    /* OBJECTIFS */
    .dom-grille{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:14px;}
    .dom-item{background:#0f1220;border:1px solid #0d1018;border-radius:12px;padding:12px 8px;text-align:center;cursor:pointer;transition:all .15s;}
    .dom-item:hover{border-color:#2a3050;}
    .dom-ic{font-size:20px;margin-bottom:4px;}
    .dom-lbl{font-size:10px;font-weight:600;color:#4a5270;text-transform:uppercase;letter-spacing:.5px;}
    .dom-cnt{font-size:11px;color:#2a3045;margin-top:2px;}

    .obj-card{background:#0f1220;border:1px solid #0d1018;border-radius:12px;padding:14px;margin-bottom:10px;transition:all .15s;}
    .obj-card:hover{border-color:#1e2435;}
    .obj-head{display:flex;align-items:flex-start;gap:10px;margin-bottom:10px;}
    .obj-dom-ic{font-size:18px;flex-shrink:0;}
    .obj-titre{font-size:14px;font-weight:600;color:#ccd0e0;flex:1;}
    .obj-badge{font-size:9px;padding:2px 8px;border-radius:20px;font-weight:600;letter-spacing:.5px;}
    .obj-prog-row{display:flex;align-items:center;gap:8px;margin-bottom:10px;}
    .obj-prog-bar{flex:1;height:4px;background:#0d1018;border-radius:2px;overflow:hidden;}
    .obj-prog-fill{height:100%;border-radius:2px;transition:width .5s;}
    .obj-pct{font-size:11px;color:#6b7280;font-weight:600;min-width:30px;text-align:right;}
    .obj-echeance{font-size:10px;color:#2a3045;margin-bottom:10px;}
    .etape{display:flex;align-items:center;gap:8px;padding:6px 0;border-bottom:1px solid #0d1018;}
    .etape:last-child{border-bottom:none;}
    .etchk{width:16px;height:16px;border-radius:4px;border:1.5px solid #1e2435;display:flex;align-items:center;justify-content:center;font-size:8px;cursor:pointer;transition:all .15s;flex-shrink:0;}
    .etchk.on{background:#6366f1;border-color:#6366f1;color:#fff;}
    .et-titre{font-size:12px;color:#8b91a8;flex:1;}
    .et-titre.fait{text-decoration:line-through;color:#2a3045;}

    /* HABITUDES */
    .hab{display:flex;flex-direction:column;gap:8px;}
    .hitem{display:flex;align-items:center;gap:10px;padding:11px 14px;background:#0f1220;border-radius:10px;border:1.5px solid transparent;cursor:pointer;transition:all .15s;}
    .hitem.fait{border-color:var(--hc);background:color-mix(in srgb,var(--hc) 8%,#0f1220);}
    .hic{font-size:19px;}
    .hn{font-size:13px;font-weight:500;color:#8b91a8;}
    .hcircle{margin-left:auto;width:22px;height:22px;border-radius:50%;border:1.5px solid #1e2435;display:flex;align-items:center;justify-content:center;font-size:10px;transition:all .15s;flex-shrink:0;}
    .hitem.fait .hcircle{background:var(--hc);border-color:var(--hc);color:#fff;}

    /* IA CHAT */
    .ia-shell{flex:1;display:flex;flex-direction:column;overflow:hidden;}
    .ia-msgs{flex:1;overflow-y:auto;padding:14px 16px;display:flex;flex-direction:column;gap:12px;}
    .msg{display:flex;flex-direction:column;gap:4px;animation:fadeUp .22s ease;}
    @keyframes fadeUp{from{opacity:0;transform:translateY(8px);}to{opacity:1;transform:translateY(0);}}
    .msg.user{align-items:flex-end;}
    .msg.assistant{align-items:flex-start;}
    .bubble{max-width:88%;padding:11px 14px;border-radius:14px;font-size:13px;line-height:1.65;}
    .msg.user .bubble{background:#6366f1;color:#fff;border-bottom-right-radius:4px;}
    .msg.assistant .bubble{background:#0f1220;color:#ccd0e0;border:1px solid #0d1018;border-bottom-left-radius:4px;}
    .bubble strong{color:#fff;font-weight:600;}
    .bubble em{color:#8b91a8;font-style:normal;background:#6366f115;padding:1px 5px;border-radius:4px;font-size:11px;}
    .sources{display:flex;flex-direction:column;gap:4px;margin-top:6px;}
    .src{display:flex;align-items:center;gap:6px;padding:5px 8px;background:#0b0e16;border-radius:7px;border:1px solid #0d1018;text-decoration:none;}
    .src-dot{width:5px;height:5px;border-radius:50%;background:#10b981;flex-shrink:0;}
    .src-lbl{font-size:10px;color:#4a5270;flex:1;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;}
    .typing{display:flex;gap:4px;padding:11px 14px;background:#0f1220;border:1px solid #0d1018;border-radius:14px;border-bottom-left-radius:4px;width:fit-content;}
    .dot{width:6px;height:6px;border-radius:50%;background:#4a5270;animation:bounce .9s infinite;}
    .dot:nth-child(2){animation-delay:.2s;}.dot:nth-child(3){animation-delay:.4s;}
    @keyframes bounce{0%,60%,100%{transform:translateY(0);}30%{transform:translateY(-5px);}}

    .sugg-row{display:flex;gap:6px;padding:8px 16px;overflow-x:auto;flex-shrink:0;border-top:1px solid #0d1018;}
    .sugg-row::-webkit-scrollbar{display:none;}
    .sugg{white-space:nowrap;padding:6px 12px;background:#0f1220;border:1px solid #0d1018;border-radius:20px;color:#4a5270;font-size:11px;cursor:pointer;font-family:'Outfit',sans-serif;transition:all .15s;flex-shrink:0;}
    .sugg:hover{border-color:#6366f1;color:#6366f1;background:#6366f108;}

    .ia-inp-zone{padding:10px 16px 10px;border-top:1px solid #0d1018;display:flex;gap:8px;align-items:flex-end;flex-shrink:0;}
    .ia-inp{flex:1;background:#0f1220;border:1px solid #0d1018;border-radius:12px;padding:10px 14px;color:#ccd0e0;font-family:'Outfit',sans-serif;font-size:13px;outline:none;resize:none;max-height:100px;line-height:1.5;transition:border .15s;}
    .ia-inp:focus{border-color:#6366f1;}
    .ia-inp::placeholder{color:#2a3045;}
    .send{width:38px;height:38px;border-radius:10px;border:none;background:#6366f1;color:#fff;cursor:pointer;font-size:16px;display:flex;align-items:center;justify-content:center;flex-shrink:0;transition:all .15s;}
    .send:hover:not(:disabled){background:#5153cc;}
    .send:disabled{opacity:.4;cursor:not-allowed;}

    /* MODAL */
    .overlay{position:fixed;inset:0;background:#000000cc;display:flex;align-items:flex-end;z-index:100;backdrop-filter:blur(6px);}
    .modal{background:#090c12;border:1px solid #0d1018;border-radius:20px 20px 0 0;padding:24px 20px 36px;width:100%;max-width:480px;margin:0 auto;max-height:90vh;overflow-y:auto;}
    .mtit{font-family:'Cormorant Garamond',serif;font-size:22px;color:#fff;margin-bottom:18px;}
    .cl{font-size:9px;font-weight:600;color:#1e2435;letter-spacing:2px;text-transform:uppercase;margin-bottom:5px;margin-top:12px;display:block;}
    .inp{width:100%;background:#0f1220;border:1px solid #0d1018;border-radius:9px;padding:10px 12px;color:#ccd0e0;font-family:'Outfit',sans-serif;font-size:13px;outline:none;transition:border .15s;}
    .inp:focus{border-color:#6366f1;}
    .row2{display:grid;grid-template-columns:1fr 1fr;gap:10px;}
    .pills{display:flex;gap:6px;flex-wrap:wrap;}
    .pill{padding:5px 12px;border-radius:20px;border:1.5px solid #0d1018;background:transparent;color:#3a4060;font-size:11px;cursor:pointer;font-family:'Outfit',sans-serif;transition:all .15s;}
    .pill.on{color:#fff;border-color:transparent;}
    .macts{display:flex;gap:10px;margin-top:18px;justify-content:flex-end;}

    /* TAB BAR */
    .tabbar{display:flex;background:#05070b;border-top:1px solid #0d1018;flex-shrink:0;}
    .tab{flex:1;display:flex;flex-direction:column;align-items:center;gap:3px;padding:10px 4px 8px;cursor:pointer;position:relative;}
    .tab-ic{font-size:16px;color:#1a2030;transition:color .15s;}
    .tab.actif .tab-ic{color:#6366f1;}
    .tab-ia{text-shadow: none; transition: all .15s;}
    .tab.actif .tab-ia{color:#6366f1;text-shadow:0 0 10px #6366f188;}
    .tab-lbl{font-size:8px;color:#1e2435;font-weight:600;letter-spacing:.5px;text-transform:uppercase;transition:color .15s;}
    .tab.actif .tab-lbl{color:#6366f1;}
    .tb{position:absolute;top:7px;right:calc(50% - 14px);background:#f43f5e;border-radius:8px;padding:1px 5px;font-size:8px;color:#fff;font-weight:700;}

    @media(min-width:640px){.app{box-shadow:0 0 60px #00000099;border-left:1px solid #0d1018;border-right:1px solid #0d1018;}}
  `;

  function renderBubble(txt){
    return txt
      .replace(/\*\*(.*?)\*\*/g,"<strong>$1</strong>")
      .replace(/`(.*?)`/g,"<em>$1</em>")
      .replace(/\n/g,"<br/>");
  }

  // ══════════════════════════════════════════════════════════
  return (
    <div className="app">
      <style>{css}</style>

      {/* HEADER */}
      <div className="hdr">
        <div>
          <div className="logo">Planify<span className="logo-ia"> ✦</span></div>
          <div className="logo-sub">{JOURS_LONGS[(TODAY.getDay()+6)%7]} {TODAY.getDate()} {MOIS[TODAY.getMonth()]} {TODAY.getFullYear()}</div>
        </div>
        {(vue==="jour"||vue==="taches") && <button className="btn-sm pri" onClick={()=>ouvrir()}>+ Ajouter</button>}
        {vue==="objectifs" && <button className="btn-sm pri" onClick={()=>setModalObj(true)}>+ Objectif</button>}
      </div>

      {/* NAV DATE */}
      {(vue==="jour"||vue==="semaine") && (
        <div className="nav-date">
          <button className="nd-btn" onClick={()=>{const d=new Date(sel);d.setDate(d.getDate()-1);setSel(d);}}>‹</button>
          <div>
            <div className="nd-txt">{JOURS_LONGS[(sel.getDay()+6)%7]} {sel.getDate()} {MOIS[sel.getMonth()]}</div>
            {memeJour(sel,TODAY) && <span className="auj-b">AUJOURD'HUI</span>}
          </div>
          <button className="nd-btn" onClick={()=>{const d=new Date(sel);d.setDate(d.getDate()+1);setSel(d);}}>›</button>
        </div>
      )}

      {/* ─── VUE IA ─── */}
      {vue==="ia" && (
        <div className="ia-shell">
          <div className="ia-msgs">
            {msgs.map((m,i)=>(
              <div key={i} className={`msg ${m.role}`}>
                <div className="bubble" dangerouslySetInnerHTML={{__html:renderBubble(m.content)}}/>
                {m.sources?.length>0 && (
                  <div className="sources">
                    {m.sources.map((s,j)=>(
                      <a key={j} className="src" href={s.url} target="_blank" rel="noreferrer">
                        <div className="src-dot"/>
                        <span className="src-lbl">🔗 {s.titre||s.url}</span>
                      </a>
                    ))}
                  </div>
                )}
              </div>
            ))}
            {loading && (
              <div className="msg assistant">
                <div className="typing"><div className="dot"/><div className="dot"/><div className="dot"/></div>
              </div>
            )}
            <div ref={endRef}/>
          </div>
          {msgs.length<=2 && (
            <div className="sugg-row">
              {SUGGESTIONS_IA.map((s,i)=>(
                <button key={i} className="sugg" onClick={()=>envoyerIA(s)}>{s}</button>
              ))}
            </div>
          )}
          <div className="ia-inp-zone">
            <textarea className="ia-inp" placeholder="Parlez-moi de vos objectifs, posez vos questions…"
              value={input} rows={1}
              onChange={e=>setInput(e.target.value)}
              onKeyDown={e=>{if(e.key==="Enter"&&!e.shiftKey){e.preventDefault();envoyerIA();}}}
            />
            <button className="send" onClick={()=>envoyerIA()} disabled={loading||!input.trim()}>
              {loading?"…":"→"}
            </button>
          </div>
        </div>
      )}

      {/* ─── VUE JOUR ─── */}
      {vue==="jour" && (
        <div className="scroll">
          <div className="stats">
            {[
              {v:`${fC}/${tJour.length}`,l:"Faites",p:tJour.length?fC/tJour.length*100:0,bg:"linear-gradient(90deg,#6366f1,#8b5cf6)"},
              {v:`${habFait}/${habs.length}`,l:"Habitudes",p:habs.length?habFait/habs.length*100:0,bg:"linear-gradient(90deg,#10b981,#34d399)"},
              {v:String(tJour.filter(t=>!t.fait).length),l:"Restantes",p:100,bg:"linear-gradient(90deg,#f43f5e,#f97316)"},
            ].map((s,i)=>(
              <div key={i} className="stat">
                <div className="sv">{s.v}</div>
                <div className="sl">{s.l}</div>
                <div className="sb"><div className="sf" style={{width:`${s.p}%`,background:s.bg}}/></div>
              </div>
            ))}
          </div>
          <div className="carte">
            <div className="ct">Planning — {JOURS_LONGS[(sel.getDay()+6)%7]} {sel.getDate()}</div>
            {HEURES.map(hr=>{
              const px=String(hr).padStart(2,"0");
              const bloc=tJour.filter(t=>t.h.startsWith(px));
              return (
                <div key={hr} className="tl">
                  <div className="tlh">{hr}h</div>
                  <div className="tlf"><div className="tlp"/></div>
                  <div className="tls">
                    {bloc.map(t=>{
                      const cat=catDe(t.cat);
                      return (
                        <div key={t.id} className={`tache ${t.fait?"fait":""}`} style={{borderLeftColor:cat.couleur}}>
                          <div className={`chk ${t.fait?"on":""}`} onClick={()=>toggle(t.id)}>{t.fait?"✓":""}</div>
                          <div style={{flex:1}}>
                            <div className="tt"><span style={{color:PRI_C[t.pri],fontSize:"9px",marginRight:"4px"}}>{PRI_I[t.pri]}</span>{t.titre}</div>
                            <div className="tm">{t.h} · {t.dur}min · <span style={{color:cat.couleur}}>{cat.label}</span>{t.notes?` · ${t.notes}`:""}</div>
                          </div>
                          <div className="acts">
                            <button className="ibtn" onClick={()=>ouvrir(t)}>✎</button>
                            <button className="ibtn" onClick={()=>del(t.id)}>✕</button>
                          </div>
                        </div>
                      );
                    })}
                    {!bloc.length&&<div style={{height:"36px"}}/>}
                  </div>
                </div>
              );
            })}
            <button className="add-btn" onClick={()=>ouvrir()}>+ Ajouter une tâche</button>
          </div>
          <div className="carte">
            <div className="ct">Habitudes du jour</div>
            <div className="hab">
              {habs.map(h=>{
                const fait=!!jrnl[`${todayStr}-${h.id}`];
                return (
                  <div key={h.id} className={`hitem ${fait?"fait":""}`} style={{"--hc":h.c}} onClick={()=>togH(h.id)}>
                    <span className="hic">{h.ic}</span>
                    <span className="hn">{h.nom}</span>
                    <div className="hcircle">{fait?"✓":""}</div>
                  </div>
                );
              })}
            </div>
          </div>
        </div>
      )}

      {/* ─── VUE SEMAINE ─── */}
      {vue==="semaine" && (
        <div className="scroll">
          <div className="carte">
            <div className="ct">Semaine du {datesSem[0].getDate()} au {datesSem[6].getDate()} {MOIS[datesSem[6].getMonth()]}</div>
            <div className="sem">
              {datesSem.map((d,i)=>{
                const dt=taches.filter(t=>t.date===d.toDateString());
                return (
                  <div key={i} className={`sd ${memeJour(d,TODAY)?"auj":""} ${memeJour(d,sel)?"sel":""}`}
                    onClick={()=>{setSel(d);setVue("jour");}}>
                    <div className="sdn">{JOURS_COURTS[i]}</div>
                    <div className="sdd">{d.getDate()}</div>
                    <div className="sdpt" style={{opacity:dt.length?1:0}}/>
                    <div className="sdc">{dt.length?`${dt.filter(t=>t.fait).length}/${dt.length}`:""}</div>
                  </div>
                );
              })}
            </div>
            {datesSem.map((d,i)=>{
              const dt=taches.filter(t=>t.date===d.toDateString());
              if(!dt.length) return null;
              return (
                <div key={i} style={{marginBottom:"14px"}}>
                  <div className="sgt">{JOURS_LONGS[i]} {d.getDate()}</div>
                  {dt.map(t=>{
                    const cat=catDe(t.cat);
                    return (
                      <div key={t.id} className="stache" style={{opacity:t.fait?.4:1}}>
                        <div className="sdot" style={{background:cat.couleur}}/>
                        <div>
                          <div className="stt" style={{textDecoration:t.fait?"line-through":"none"}}>{t.titre}</div>
                          <div className="ssb">{t.h} · {cat.label}</div>
                        </div>
                      </div>
                    );
                  })}
                </div>
              );
            })}
          </div>
        </div>
      )}

      {/* ─── VUE OBJECTIFS ─── */}
      {vue==="objectifs" && (
        <div className="scroll">
          <div className="dom-grille">
            {DOMAINES.map(d=>{
              const cnt=objectifs.filter(o=>o.domaine===d.id).length;
              const moy=cnt?Math.round(objectifs.filter(o=>o.domaine===d.id).reduce((s,o)=>s+o.progres,0)/cnt):0;
              return (
                <div key={d.id} className="dom-item" style={{borderColor:cnt?"#1e2435":"transparent"}}>
                  <div className="dom-ic">{d.ic}</div>
                  <div className="dom-lbl" style={{color:d.couleur}}>{d.label}</div>
                  <div className="dom-cnt">{cnt} objectif{cnt!==1?"s":""}{cnt?` · ${moy}%`:""}</div>
                </div>
              );
            })}
          </div>
          {objectifs.map(o=>{
            const dom=domaineDe(o.domaine);
            const etapesFaites=o.etapes.filter(e=>e.fait).length;
            return (
              <div key={o.id} className="obj-card">
                <div className="obj-head">
                  <span className="obj-dom-ic">{dom.ic}</span>
                  <span className="obj-titre">{o.titre}</span>
                  <span className="obj-badge" style={{background:`${dom.couleur}20`,color:dom.couleur}}>{PRI_I[o.priorite]}</span>
                </div>
                <div className="obj-prog-row">
                  <div className="obj-prog-bar">
                    <div className="obj-prog-fill" style={{width:`${o.progres}%`,background:`linear-gradient(90deg,${dom.couleur},${dom.couleur}aa)`}}/>
                  </div>
                  <span className="obj-pct">{o.progres}%</span>
                </div>
                {o.echeance && <div className="obj-echeance">📅 Échéance : {new Date(o.echeance).toLocaleDateString("fr-FR",{day:"numeric",month:"long",year:"numeric"})}</div>}
                <div style={{marginTop:"2px"}}>
                  {o.etapes.map(e=>(
                    <div key={e.id} className="etape">
                      <div className={`etchk ${e.fait?"on":""}`} onClick={()=>togEtape(o.id,e.id)}>{e.fait?"✓":""}</div>
                      <span className={`et-titre ${e.fait?"fait":""}`}>{e.titre}</span>
                    </div>
                  ))}
                </div>
                <div style={{marginTop:"10px",fontSize:"10px",color:"#2a3045"}}>{etapesFaites}/{o.etapes.length} étapes · <span style={{color:"#6366f1",cursor:"pointer"}} onClick={()=>envoyerIA(`Aide-moi à avancer sur mon objectif : ${o.titre}`)}>Demander à l'IA →</span></div>
              </div>
            );
          })}
          {objectifs.length===0 && (
            <div style={{textAlign:"center",padding:"30px",color:"#1e2435",fontSize:"13px"}}>
              <div style={{fontSize:"28px",marginBottom:"8px"}}>🎯</div>
              Aucun objectif pour l'instant.<br/>Demandez à l'IA de vous aider à en définir !
            </div>
          )}
        </div>
      )}

      {/* ─── VUE TÂCHES ─── */}
      {vue==="taches" && (
        <div className="scroll">
          {CATEGORIES.map(cat=>{
            const ct=taches.filter(t=>t.cat===cat.id);
            if(!ct.length) return null;
            return (
              <div key={cat.id} className="carte">
                <div style={{display:"flex",alignItems:"center",gap:"8px",marginBottom:"12px"}}>
                  <div style={{width:"7px",height:"7px",borderRadius:"50%",background:cat.couleur}}/>
                  <span style={{fontSize:"11px",fontWeight:600,color:cat.couleur,letterSpacing:"1px",textTransform:"uppercase",flex:1}}>{cat.label}</span>
                  <span style={{fontSize:"10px",color:"#1e2435"}}>{ct.filter(t=>t.fait).length}/{ct.length}</span>
                </div>
                {ct.map(t=>(
                  <div key={t.id} className={`tache ${t.fait?"fait":""}`} style={{borderLeftColor:cat.couleur,marginBottom:"6px"}}>
                    <div className={`chk ${t.fait?"on":""}`} onClick={()=>toggle(t.id)}>{t.fait?"✓":""}</div>
                    <div style={{flex:1}}>
                      <div className="tt"><span style={{color:PRI_C[t.pri],fontSize:"9px",marginRight:"4px"}}>{PRI_I[t.pri]}</span>{t.titre}</div>
                      <div className="tm">{new Date(t.date).toLocaleDateString("fr-FR",{day:"numeric",month:"short"})} à {t.h} · {t.dur}min</div>
                    </div>
                    <div className="acts">
                      <button className="ibtn" onClick={()=>ouvrir(t)}>✎</button>
                      <button className="ibtn" onClick={()=>del(t.id)}>✕</button>
                    </div>
                  </div>
                ))}
              </div>
            );
          })}
        </div>
      )}

      {/* TAB BAR */}
      <div className="tabbar">
        {NAV.map(n=>{
          const actif=vue===n.id;
          const badge=n.id==="taches"?taches.filter(t=>!t.fait).length:0;
          return (
            <div key={n.id} className={`tab ${actif?"actif":""}`} onClick={()=>setVue(n.id)}>
              <span className={`tab-ic ${n.id==="ia"?"tab-ia":""}`}>{n.ic}</span>
              <span className="tab-lbl">{n.label}</span>
              {badge>0&&!actif&&<span className="tb">{badge}</span>}
            </div>
          );
        })}
      </div>

      {/* MODAL TÂCHE */}
      {modal && (
        <div className="overlay" onClick={e=>e.target===e.currentTarget&&setModal(false)}>
          <div className="modal">
            <div className="mtit">{editT?"Modifier":"Nouvelle tâche"}</div>
            <label className="cl">Titre</label>
            <input className="inp" placeholder="Nom de la tâche..." value={form.titre} onChange={e=>setForm({...form,titre:e.target.value})}/>
            <div className="row2">
              <div><label className="cl">Heure</label><input className="inp" type="time" value={form.h} onChange={e=>setForm({...form,h:e.target.value})}/></div>
              <div><label className="cl">Durée (min)</label><input className="inp" type="number" min="5" step="5" value={form.dur} onChange={e=>setForm({...form,dur:Number(e.target.value)})}/></div>
            </div>
            <label className="cl">Catégorie</label>
            <div className="pills">
              {CATEGORIES.map(c=>(
                <button key={c.id} className={`pill ${form.cat===c.id?"on":""}`}
                  style={form.cat===c.id?{background:c.couleur}:{}} onClick={()=>setForm({...form,cat:c.id})}>{c.label}</button>
              ))}
            </div>
            <label className="cl">Priorité</label>
            <div className="pills">
              {[["haute","🔴 Haute"],["moyenne","🟡 Moyenne"],["basse","🟢 Basse"]].map(([p,l])=>(
                <button key={p} className={`pill ${form.pri===p?"on":""}`}
                  style={form.pri===p?{background:"#1a1f30",borderColor:"#6366f1"}:{}} onClick={()=>setForm({...form,pri:p})}>{l}</button>
              ))}
            </div>
            <label className="cl">Notes</label>
            <input className="inp" placeholder="Notes optionnelles..." value={form.notes} onChange={e=>setForm({...form,notes:e.target.value})}/>
            <div className="macts">
              <button className="btn-sm" onClick={()=>setModal(false)}>Annuler</button>
              <button className="btn-sm pri" onClick={sauver}>Enregistrer</button>
            </div>
          </div>
        </div>
      )}

      {/* MODAL OBJECTIF */}
      {modalObj && (
        <div className="overlay" onClick={e=>e.target===e.currentTarget&&setModalObj(false)}>
          <div className="modal">
            <div className="mtit">Nouvel objectif</div>
            <label className="cl">Titre de l'objectif</label>
            <input className="inp" placeholder="Ex: Lancer mon business..." value={formObj.titre} onChange={e=>setFormObj({...formObj,titre:e.target.value})}/>
            <label className="cl">Domaine de vie</label>
            <div className="pills">
              {DOMAINES.map(d=>(
                <button key={d.id} className={`pill ${formObj.domaine===d.id?"on":""}`}
                  style={formObj.domaine===d.id?{background:d.couleur}:{}} onClick={()=>setFormObj({...formObj,domaine:d.id})}>
                  {d.ic} {d.label}
                </button>
              ))}
            </div>
            <label className="cl">Échéance</label>
            <input className="inp" type="date" value={formObj.echeance} onChange={e=>setFormObj({...formObj,echeance:e.target.value})}/>
            <label className="cl">Priorité</label>
            <div className="pills">
              {[["haute","🔴 Haute"],["moyenne","🟡 Moyenne"],["basse","🟢 Basse"]].map(([p,l])=>(
                <button key={p} className={`pill ${formObj.priorite===p?"on":""}`}
                  style={formObj.priorite===p?{background:"#1a1f30",borderColor:"#6366f1"}:{}} onClick={()=>setFormObj({...formObj,priorite:p})}>{l}</button>
              ))}
            </div>
            <div className="macts">
              <button className="btn-sm" onClick={()=>setModalObj(false)}>Annuler</button>
              <button className="btn-sm pri" onClick={ajouterObj}>Créer l'objectif</button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
