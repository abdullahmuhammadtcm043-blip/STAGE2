import React, { useState, useEffect, useRef } from 'react';
import { 
  BookOpen, Video, FileText, Bell, 
  MessageCircle, Settings, LogOut, Home, User, 
  Plus, Trash2, ShieldCheck,
  GraduationCap, Send, Eye, Lock,
  Sparkles, BrainCircuit, Download, Youtube,
  FileSpreadsheet, FileCode, Users, Key, EyeOff, Search,
  Database, Zap, Info, Globe, Layout
} from 'lucide-react';

// API Configuration
const apiKey = ""; 
const GEMINI_MODEL = "gemini-2.5-flash-preview-09-2025";

// Admin & Owner Configuration
const OWNER_NAME = "عبدالله مشتاق";
const OWNER_PASS = "2005822";

const ADMIN_CONFIG = {
  'حسن تركي': { role: 'مشرف', section: 'A' },
  'يوسف نورس': { role: 'مشرف', section: 'B' },
  'صاحب نعمة': { role: 'مشرف', section: 'مسائي' }
};

const STORAGE_KEYS = {
  USERS: 'stage2_users_v6',
  SUBJECTS: 'stage2_subjects_v6',
  MATERIALS: 'stage2_materials_v6',
  SECTION_CODES: 'stage2_section_codes_v6',
  CHATS: 'stage2_global_chats_v6',
  CURRENT_PROFILE: 'stage2_current_profile_v6'
};

export default function App() {
  const [profile, setProfile] = useState(null);
  const [view, setView] = useState('home'); 
  const [activeSubject, setActiveSubject] = useState(null);
  const [activeMaterial, setActiveMaterial] = useState(null);
  const [toast, setToast] = useState(null);
  const [aiLoading, setAiLoading] = useState(false);
  const [showSectionModal, setShowSectionModal] = useState(null);
  
  const [subjects, setSubjects] = useState([]);
  const [materials, setMaterials] = useState([]);
  const [sectionCodes, setSectionCodes] = useState({ 'A': '', 'B': '', 'C': '', 'مسائي': '' });
  const [globalChats, setGlobalChats] = useState([]);

  // Load Data on Start
  useEffect(() => {
    const loadData = () => {
      const storedSubjects = JSON.parse(localStorage.getItem(STORAGE_KEYS.SUBJECTS) || '[]');
      const storedMaterials = JSON.parse(localStorage.getItem(STORAGE_KEYS.MATERIALS) || '[]');
      const storedChats = JSON.parse(localStorage.getItem(STORAGE_KEYS.CHATS) || '[]');
      const storedCodes = JSON.parse(localStorage.getItem(STORAGE_KEYS.SECTION_CODES) || '{"A":"","B":"","C":"","مسائي":""}');
      
      setSubjects(storedSubjects);
      setMaterials(storedMaterials);
      setGlobalChats(storedChats);
      setSectionCodes(storedCodes);

      const savedProfile = localStorage.getItem(STORAGE_KEYS.CURRENT_PROFILE);
      if (savedProfile) {
        setProfile(JSON.parse(savedProfile));
      }
    };
    loadData();
  }, []);

  const showToast = (message, type = 'success') => {
    setToast({ message, type });
    setTimeout(() => setToast(null), 3000);
  };

  const isOwner = profile && profile.username === OWNER_NAME;
  const isSupervisor = profile && (ADMIN_CONFIG[profile.username] || isOwner);

  // AI Assistant Logic
  const askGeminiWithContext = async (prompt) => {
    setAiLoading(true);
    const subjectsContext = subjects.map(s => `${s.name} (${s.section})`).join(', ');
    const knowledgeBase = `سياق المرحلة الثانية: المواد المتاحة هي ${subjectsContext}.`;

    try {
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/${GEMINI_MODEL}:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: `${knowledgeBase}\n\nسؤال المستخدم: ${prompt}` }] }],
          tools: [{ "google_search": {} }],
          systemInstruction: { 
            parts: [{ text: "أنت المساعد الذكي STAGE2. ابحث في جوجل وقدم إجابة دقيقة لطلبة الهندسة/المرحلة الثانية. كن مختصراً." }] 
          }
        })
      });

      if (!response.ok) throw new Error("Search Engine Error");
      
      const result = await response.json();
      setAiLoading(false);
      return result.candidates?.[0]?.content?.parts?.[0]?.text || "اعتذر، لم أتمكن من العثور على إجابة.";
    } catch (error) {
      setAiLoading(false);
      return "حدث خطأ أثناء الاتصال بمحرك البحث. يرجى المحاولة لاحقاً.";
    }
  };

  const Navigation = () => (
    <nav className="bg-white/80 backdrop-blur-md text-slate-800 p-4 sticky top-0 z-50 shadow-sm flex justify-between items-center border-b border-slate-100" dir="rtl">
      <div className="flex items-center gap-3 cursor-pointer" onClick={() => setView('home')}>
        <div className="bg-gradient-to-br from-indigo-500 to-purple-500 p-2 rounded-xl text-white">
          <Zap size={20} fill="currentColor"/>
        </div>
        <span className="font-black text-xl tracking-tight text-slate-900">STAGE2</span>
      </div>
      
      <div className="flex gap-2 items-center">
        {profile ? (
          <>
            <button onClick={() => setView('home')} className="p-2.5 text-slate-400 hover:text-indigo-600 rounded-xl transition-all"><Home size={20}/></button>
            <button onClick={() => setView('profile')} className="p-2.5 text-slate-400 hover:text-indigo-600 rounded-xl transition-all"><User size={20}/></button>
            <button onClick={() => setView('meet')} className="flex p-2 px-4 bg-indigo-600 text-white rounded-xl font-bold items-center gap-2 hover:bg-indigo-700 transition-all text-xs">
              <MessageCircle size={16}/> الميت الذكي
            </button>
            <button onClick={() => { localStorage.removeItem(STORAGE_KEYS.CURRENT_PROFILE); setProfile(null); setView('auth'); }} className="p-2.5 text-slate-400 hover:text-red-500 rounded-xl transition-all"><LogOut size={20}/></button>
          </>
        ) : (
          <button onClick={() => setView('auth')} className="bg-indigo-600 text-white px-6 py-2 rounded-xl font-bold text-sm">دخول</button>
        )}
      </div>
    </nav>
  );

  const HomeView = () => (
    <div className="max-w-6xl mx-auto p-6 md:p-12 text-center" dir="rtl">
      <div className="mb-16">
        <div className="inline-flex items-center gap-2 bg-indigo-50 text-indigo-600 px-4 py-1.5 rounded-full text-[10px] font-black mb-6 border border-indigo-100 uppercase">
          <Globe size={12}/> متصل بمحرك Google Search
        </div>
        <h1 className="text-4xl md:text-6xl font-black text-slate-900 mb-6 leading-tight">منصة المرحلة الثانية <br/><span className="text-indigo-600">بذكاء عبدالله مشتاق</span></h1>
        <p className="text-slate-500 font-medium max-w-lg mx-auto text-lg mb-10">إدارة المواد، المحاضرات، والمناقشات الذكية في مكان واحد.</p>
        
        <div className="flex flex-wrap justify-center gap-4">
          <button onClick={() => setView('subjects_list')} className="bg-indigo-600 text-white px-10 py-4 rounded-2xl font-black shadow-xl hover:bg-slate-900 transition-all">
             استعراض المواد
          </button>
          {isOwner && (
             <button onClick={() => setView('meet')} className="bg-slate-900 text-white px-10 py-4 rounded-2xl font-black shadow-xl hover:bg-indigo-600 transition-all">
                غرفة العمليات الذكية
             </button>
          )}
        </div>
      </div>
      
      {isOwner && (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-20">
          {['A', 'B', 'C', 'مسائي'].map(s => (
            <div key={s} className="bg-white p-8 rounded-[2.5rem] shadow-sm border border-slate-100 hover:shadow-xl transition-all cursor-pointer"
                 onClick={() => setShowSectionModal(s)}>
              <div className="w-14 h-14 bg-indigo-50 text-indigo-400 rounded-2xl flex items-center justify-center mx-auto mb-6">
                <Users size={24} />
              </div>
              <h2 className="text-lg font-black text-slate-800">شعبة {s}</h2>
              <p className="text-[10px] text-slate-400 font-bold mt-2 italic">إدارة الرموز والمواد</p>
            </div>
          ))}
        </div>
      )}
    </div>
  );

  const ProfileView = () => (
    <div className="max-w-2xl mx-auto p-6 md:p-12" dir="rtl">
       <div className="bg-white rounded-[3.5rem] shadow-2xl border border-slate-50 overflow-hidden">
          <div className="h-32 bg-indigo-600"></div>
          <div className="px-10 pb-12 -mt-16 text-center">
             <div className="w-32 h-32 bg-white rounded-[2.5rem] p-2 mx-auto shadow-xl">
                <div className="w-full h-full bg-slate-50 rounded-[2rem] flex items-center justify-center text-slate-300">
                   <User size={64}/>
                </div>
             </div>
             <h2 className="text-3xl font-black mt-6 text-slate-900">{profile.username}</h2>
             <p className="text-indigo-500 font-bold text-sm mb-8">{isOwner ? 'المالك العام' : `طالب - شعبة ${profile.section}`}</p>
             
             <div className="grid grid-cols-2 gap-4 text-right">
                <div className="bg-slate-50 p-6 rounded-3xl">
                   <p className="text-[10px] font-black text-slate-400 uppercase mb-2">الشعبة</p>
                   <p className="font-black text-xl text-slate-800">{profile.section}</p>
                </div>
                <div className="bg-slate-50 p-6 rounded-3xl">
                   <p className="text-[10px] font-black text-slate-400 uppercase mb-2">الصلاحية</p>
                   <p className="font-black text-xl text-slate-800">{isOwner ? 'كاملة' : 'مستخدم'}</p>
                </div>
             </div>
             <button onClick={() => setView('home')} className="mt-10 w-full bg-slate-900 text-white p-5 rounded-2xl font-black shadow-lg">العودة للرئيسية</button>
          </div>
       </div>
    </div>
  );

  const AuthView = () => {
    const [u, setU] = useState('');
    const [p, setP] = useState('');
    const [s, setS] = useState('A');
    const [isReg, setIsReg] = useState(false);

    const handle = () => {
      if (u === OWNER_NAME && p === OWNER_PASS) {
        const ownerProfile = { username: OWNER_NAME, section: 'ALL' };
        setProfile(ownerProfile);
        localStorage.setItem(STORAGE_KEYS.CURRENT_PROFILE, JSON.stringify(ownerProfile));
        setView('home');
        showToast("مرحباً أونر عبدالله مشتاق");
        return;
      }

      const users = JSON.parse(localStorage.getItem(STORAGE_KEYS.USERS) || '[]');
      if(isReg) {
        if(users.find(x=>x.username === u)) return showToast("الاسم مسجل مسبقاً", "error");
        const n = {username: u, password: p, section: s};
        const newUsers = [...users, n];
        localStorage.setItem(STORAGE_KEYS.USERS, JSON.stringify(newUsers));
        setProfile(n);
        localStorage.setItem(STORAGE_KEYS.CURRENT_PROFILE, JSON.stringify(n));
        setView('subjects_list');
      } else {
        const found = users.find(x=>x.username === u && x.password === p);
        if(found) { 
          setProfile(found); 
          localStorage.setItem(STORAGE_KEYS.CURRENT_PROFILE, JSON.stringify(found)); 
          setView('subjects_list');
        }
        else showToast("تأكد من البيانات", "error");
      }
    };

    return (
      <div className="min-h-[80vh] flex items-center justify-center p-6" dir="rtl">
        <div className="max-w-md w-full bg-white p-12 rounded-[3.5rem] shadow-2xl">
           <div className="w-16 h-16 bg-indigo-600 text-white rounded-[1.5rem] flex items-center justify-center mx-auto mb-8"><GraduationCap size={32}/></div>
           <h2 className="text-3xl font-black mb-6 text-center text-slate-800">{isReg ? 'إنشاء حساب' : 'تسجيل دخول'}</h2>
           <div className="space-y-4">
              <input value={u} onChange={e=>setU(e.target.value)} placeholder="الاسم الثنائي" className="w-full p-4 bg-slate-50 rounded-2xl font-bold outline-none text-center" />
              <input value={p} onChange={e=>setP(e.target.value)} type="password" placeholder="كلمة المرور" className="w-full p-4 bg-slate-50 rounded-2xl font-bold outline-none text-center" />
              {isReg && (
                <select value={s} onChange={e=>setS(e.target.value)} className="w-full p-4 bg-slate-50 rounded-2xl font-bold outline-none text-center">
                    <option value="A">شعبة A</option><option value="B">شعبة B</option><option value="C">شعبة C</option><option value="مسائي">مسائي</option>
                </select>
              )}
              <button onClick={handle} className="w-full bg-indigo-600 text-white p-5 rounded-2xl font-black shadow-xl hover:bg-slate-900 transition-all">دخول</button>
              <button onClick={()=>setIsReg(!isReg)} className="w-full text-slate-400 font-bold text-xs mt-6">{isReg ? 'لديك حساب؟ سجل دخول' : 'ليس لديك حساب؟ سجل الآن'}</button>
           </div>
        </div>
      </div>
    );
  };

  return (
    <div className="min-h-screen bg-[#FDFDFF] pb-20 font-sans">
      {toast && <div className="fixed bottom-10 right-10 z-[100] px-8 py-4 bg-slate-900 text-white rounded-2xl shadow-2xl font-black">{toast.message}</div>}
      
      {showSectionModal && (
        <div className="fixed inset-0 z-[70] bg-slate-900/40 backdrop-blur-md flex items-center justify-center p-4">
           <div className="bg-white p-10 rounded-[3rem] shadow-2xl text-center w-full max-w-sm">
              <h3 className="text-2xl font-black mb-6">رمز الدخول ({showSectionModal})</h3>
              <input id="vCode" type="password" placeholder="••••••" className="w-full p-5 bg-slate-50 rounded-2xl mb-6 font-black text-center text-2xl outline-none" />
              <div className="flex gap-3">
                <button onClick={() => {
                  const val = document.getElementById('vCode').value;
                  if(val === sectionCodes[showSectionModal] || isOwner) { setView('subjects_list'); setShowSectionModal(null); }
                  else showToast("الرمز خطأ", "error");
                }} className="flex-grow bg-indigo-600 text-white p-5 rounded-2xl font-black">دخول</button>
                <button onClick={()=>setShowSectionModal(null)} className="px-6 bg-slate-100 text-slate-500 rounded-2xl font-bold">إلغاء</button>
              </div>
           </div>
        </div>
      )}

      {activeMaterial && <MaterialViewer material={activeMaterial} onClose={()=>setActiveMaterial(null)} askGemini={askGeminiWithContext} aiLoading={aiLoading} />}

      <Navigation />

      <main>
        {!profile ? <AuthView /> : (
          <>
            {view === 'home' && <HomeView />}
            {view === 'profile' && <ProfileView />}
            {view === 'subjects_list' && <Subjects_List subjects={subjects} setSubjects={setSubjects} isOwner={isOwner} isSupervisor={isSupervisor} profile={profile} setView={setView} setActiveSubject={setActiveSubject} />}
            {view === 'subject_materials' && <Subject_Materials subject={activeSubject} materials={materials} setMaterials={setMaterials} isSupervisor={isSupervisor} profile={profile} setActiveMaterial={setActiveMaterial} setView={setView} />}
            {view === 'meet' && <StudyMeet askGemini={askGeminiWithContext} profile={profile} isOwner={isOwner} setGlobalChats={setGlobalChats} l={aiLoading} setL={setAiLoading}/>}
          </>
        )}
      </main>
    </div>
  );
}

function MaterialViewer({ material, onClose, askGemini, aiLoading }) {
    const [q, setQ] = useState('');
    const [res, setRes] = useState('');
    const isYoutube = material.url.includes('youtube') || material.url.includes('youtu.be');

    return (
      <div className="fixed inset-0 z-[60] bg-slate-900/95 backdrop-blur-md flex flex-col md:flex-row" dir="rtl">
        <div className="flex-grow p-4 md:p-8 flex flex-col">
          <div className="flex justify-between items-center text-white mb-6 px-4">
             <h2 className="text-xl font-black">{material.title}</h2>
             <button onClick={onClose} className="bg-white/10 p-3 rounded-full hover:bg-red-500"><LogOut size={20}/></button>
          </div>
          <div className="flex-grow bg-white rounded-[2.5rem] overflow-hidden shadow-2xl relative">
            {isYoutube ? (
              <iframe className="w-full h-full" src={material.url.replace('watch?v=', 'embed/')} frameBorder="0" allowFullScreen></iframe>
            ) : (
              <div className="w-full h-full flex flex-col items-center justify-center p-12 text-center bg-slate-50">
                 <FileText size={64} className="text-indigo-500 mb-6"/>
                 <h3 className="text-xl font-black text-slate-800 mb-6">المحاضرة جاهزة للعرض</h3>
                 <a href={material.url} target="_blank" rel="noreferrer" className="bg-indigo-600 text-white px-10 py-4 rounded-2xl font-black flex items-center gap-2">
                   <Download size={20}/> فتح الملف
                 </a>
              </div>
            )}
          </div>
        </div>
        <div className="w-full md:w-[400px] bg-white flex flex-col m-4 rounded-[2.5rem] overflow-hidden border border-slate-100">
          <div className="p-6 border-b font-black text-indigo-900 bg-indigo-50/30 flex items-center gap-2">
             <Sparkles size={18}/> المحلل الذكي STAGE2
          </div>
          <div className="flex-grow overflow-y-auto p-6 space-y-4">
            {res && <div className="bg-slate-50 p-6 rounded-[2rem] text-slate-700 text-sm leading-relaxed border border-slate-100">{res}</div>}
            {aiLoading && <div className="text-center py-20 text-indigo-400 font-bold animate-pulse">جارٍ البحث والتحليل...</div>}
            {!res && !aiLoading && <p className="text-center py-20 text-slate-300 font-bold">اسأل الذكاء الاصطناعي عن المحاضرة</p>}
          </div>
          <form className="p-4 border-t bg-slate-50 flex gap-2" onSubmit={async e => {
            e.preventDefault();
            if(!q.trim()) return;
            const r = await askGemini(q);
            setRes(r);
            setQ('');
          }}>
            <input value={q} onChange={e=>setQ(e.target.value)} placeholder="اسأل أي شيء..." className="flex-grow p-4 bg-white rounded-2xl font-bold text-sm outline-none border border-slate-200" />
            <button className="bg-indigo-600 text-white p-4 rounded-2xl"><Send size={20}/></button>
          </form>
        </div>
      </div>
    );
}

function Subjects_List({ subjects, setSubjects, isOwner, isSupervisor, profile, setView, setActiveSubject }) {
  const mySubjects = subjects.filter(s => isOwner || s.section === profile.section);

  const addSubject = () => {
    const n = prompt("اسم المادة الجديدة:");
    if(!n) return;
    const sectionToAdd = isOwner ? 'A' : (ADMIN_CONFIG[profile.username]?.section || profile.section);
    const updated = [...subjects, {id: Date.now(), name: n, section: sectionToAdd}];
    setSubjects(updated);
    localStorage.setItem(STORAGE_KEYS.SUBJECTS, JSON.stringify(updated));
  };

  const deleteSubject = (id, e) => {
    e.stopPropagation();
    if(!window.confirm("حذف المادة؟")) return;
    const updated = subjects.filter(s => s.id !== id);
    setSubjects(updated);
    localStorage.setItem(STORAGE_KEYS.SUBJECTS, JSON.stringify(updated));
  };

  return (
    <div className="max-w-6xl mx-auto p-10" dir="rtl">
       <div className="flex justify-between items-center mb-12">
          <h1 className="text-3xl font-black text-slate-900">المواد الدراسية (شعبة {profile.section})</h1>
          {isSupervisor && (
            <button onClick={addSubject} className="bg-indigo-600 text-white px-8 py-4 rounded-2xl font-black flex items-center gap-2 shadow-xl"><Plus size={20}/> إضافة مادة</button>
          )}
       </div>
       <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
          {mySubjects.map(s => (
            <div key={s.id} onClick={()=>{setActiveSubject(s); setView('subject_materials');}} className="bg-white p-10 rounded-[3rem] shadow-sm border border-slate-100 hover:shadow-2xl transition-all cursor-pointer relative group">
               {isSupervisor && (
                 <button onClick={(e)=>deleteSubject(s.id, e)} className="absolute top-6 left-6 text-slate-200 hover:text-red-500 opacity-0 group-hover:opacity-100 transition-all"><Trash2 size={18}/></button>
               )}
               <div className="w-16 h-16 bg-indigo-50 text-indigo-500 rounded-2xl flex items-center justify-center mb-6"><BookOpen size={32}/></div>
               <h3 className="text-xl font-black text-slate-800">{s.name}</h3>
               <p className="text-slate-400 text-xs font-bold mt-2">استكشاف المحاضرات</p>
            </div>
          ))}
       </div>
    </div>
  );
}

function Subject_Materials({ subject, materials, setMaterials, isSupervisor, profile, setActiveMaterial, setView }) {
  const mats = materials.filter(m => m.subjectId === subject.id);
  
  const addMaterial = () => {
     const t = prompt("عنوان المحاضرة:");
     const u = prompt("الرابط (يوتيوب أو ملف):");
     if(!t || !u) return;
     const updated = [...materials, {id: Date.now(), subjectId: subject.id, title: t, url: u}];
     setMaterials(updated);
     localStorage.setItem(STORAGE_KEYS.MATERIALS, JSON.stringify(updated));
  };

  const deleteMaterial = (id, e) => {
    e.stopPropagation();
    const updated = materials.filter(m => m.id !== id);
    setMaterials(updated);
    localStorage.setItem(STORAGE_KEYS.MATERIALS, JSON.stringify(updated));
  };

  return (
    <div className="max-w-5xl mx-auto p-10" dir="rtl">
       <button onClick={()=>setView('subjects_list')} className="mb-8 font-black text-slate-400 hover:text-indigo-600 flex items-center gap-2 text-sm">
          <Home size={16}/> العودة للمواد
       </button>
       <div className="flex justify-between items-center mb-10">
          <h1 className="text-4xl font-black text-slate-900">{subject.name}</h1>
          {isSupervisor && (
             <button onClick={addMaterial} className="bg-indigo-600 text-white px-8 py-4 rounded-2xl font-black shadow-lg hover:bg-slate-900">نشر محاضرة</button>
          )}
       </div>

       <div className="grid gap-4">
          {mats.map(m => (
            <div key={m.id} onClick={()=>setActiveMaterial(m)} className="p-6 bg-white rounded-[2.5rem] shadow-sm flex justify-between items-center cursor-pointer hover:scale-[1.01] transition-all group">
               <div className="flex items-center gap-5">
                  <div className={`p-4 rounded-2xl ${m.url.includes('youtube') ? 'bg-red-50 text-red-500' : 'bg-indigo-50 text-indigo-500'}`}>
                    {m.url.includes('youtube') ? <Youtube size={22}/> : <FileText size={22}/>}
                  </div>
                  <span className="font-black text-lg text-slate-800">{m.title}</span>
               </div>
               <div className="flex items-center gap-4">
                  {isSupervisor && (
                    <button onClick={(e)=>deleteMaterial(m.id, e)} className="p-2 text-slate-200 hover:text-red-500"><Trash2 size={20}/></button>
                  )}
                  <div className="bg-slate-50 p-3 rounded-full text-slate-300 group-hover:bg-indigo-600 group-hover:text-white transition-all"><Eye size={20}/></div>
               </div>
            </div>
          ))}
       </div>
    </div>
  );
}

function StudyMeet({ askGemini, profile, isOwner, setGlobalChats, l, setL }) {
  const [msgs, setMsgs] = useState([]);
  const [inp, setInp] = useState('');

  useEffect(() => {
    const saved = JSON.parse(localStorage.getItem(STORAGE_KEYS.CHATS) || '[]');
    setMsgs(saved);
  }, []);

  const send = async e => {
    e.preventDefault();
    if(!inp.trim()) return;
    const n = { sender: profile.username, text: inp, time: new Date().toLocaleTimeString() };
    const updated = [...msgs, n];
    setMsgs(updated);
    setGlobalChats(updated);
    localStorage.setItem(STORAGE_KEYS.CHATS, JSON.stringify(updated));
    const cur = inp;
    setInp('');

    if(cur.includes('@ذكاء')) {
      setL(true);
      const res = await askGemini(cur.replace('@ذكاء', ''));
      const aiMsg = { sender: 'STAGE2 AI ✨', text: res, isAi: true, time: new Date().toLocaleTimeString() };
      const final = [...updated, aiMsg];
      setMsgs(final);
      setGlobalChats(final);
      localStorage.setItem(STORAGE_KEYS.CHATS, JSON.stringify(final));
      setL(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6 md:p-10 h-[85vh] flex flex-col" dir="rtl">
       <div className="bg-white p-8 rounded-t-[3rem] flex justify-between items-center shadow-sm border border-slate-100">
          <h2 className="text-2xl font-black flex items-center gap-3"><MessageCircle className="text-indigo-600"/> الميت الدراسي الذكي</h2>
          <div className="bg-green-50 text-green-600 px-4 py-1.5 rounded-full text-[10px] font-black border border-green-100">Live Chat</div>
       </div>
       <div className="flex-grow bg-[#F8F9FF] border-x overflow-y-auto p-8 space-y-6 shadow-inner">
          {msgs.map((m, i) => (
            <div key={i} className={`flex flex-col ${m.sender === profile.username ? 'items-start' : 'items-end'}`}>
               <span className="text-[9px] font-black text-slate-400 mb-1 px-3 uppercase">{m.sender}</span>
               <div className={`p-5 rounded-[2rem] max-w-[85%] shadow-sm ${m.isAi ? 'bg-indigo-600 text-white rounded-tr-none' : (m.sender === profile.username ? 'bg-slate-900 text-white rounded-tr-none' : 'bg-white text-slate-800 border border-slate-100 rounded-tl-none')}`}>
                  <p className="text-sm font-bold leading-relaxed">{m.text}</p>
               </div>
            </div>
          ))}
          {l && <div className="text-[10px] font-black text-indigo-500 animate-pulse px-4">جارٍ البحث والتحليل...</div>}
       </div>
       <form onSubmit={send} className="bg-white p-4 border rounded-b-[3rem] flex gap-3 shadow-xl">
          <input value={inp} onChange={e=>setInp(e.target.value)} placeholder="اكتب سؤالك.. منشن @ذكاء للبحث" className="flex-grow p-5 bg-slate-50 rounded-[2rem] font-bold outline-none border-2 border-transparent focus:border-indigo-500 transition-all shadow-inner" />
          <button className="bg-indigo-600 text-white p-5 rounded-full shadow-lg hover:scale-110 active:scale-90 transition-all"><Send size={24}/></button>
       </form>
    </div>
  );
}
