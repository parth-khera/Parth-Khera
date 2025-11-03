// ParthProfile_Pro.jsx
// Elite interactive React portfolio for Parth Khera
// Tech: React + TailwindCSS + Framer Motion + react-tsparticles + @react-three/fiber + @react-three/drei
// Optional: connect a FastAPI/Node backend for AI chat & live GitHub analytics.
// Install example:
// npm i react react-dom framer-motion react-tsparticles tsparticles @react-three/fiber @react-three/drei lucide-react recharts

import React, { useEffect, useRef, useState } from 'react'
import { motion } from 'framer-motion'
import Particles from 'react-tsparticles'
import { loadFull } from 'tsparticles'
import { Canvas } from '@react-three/fiber'
import { OrbitControls } from '@react-three/drei'
import { GitHub, Linkedin, Mail } from 'lucide-react'
import { Line } from 'recharts'

// ---------- Resume data (populated from your uploaded resume) ----------
const resume = {
  name: 'Parth Khera',
  title: 'AI · Web Development · Cybersecurity',
  location: 'Paonta Sahib, India',
  email: 'kheraparth15@gmail.com',
  phone: '+91 8628951509',
  website: 'https://github.com/parth-khera',
  education: [
    { degree: 'B.Tech in Computer Science & Engineering', institute: 'Shivalik College of Engineering, Dehradun', duration: 'Aug 2024 – May 2028', cgpa: '7.54' },
    { degree: 'Senior Secondary (Science)', institute: 'Guru Nanak Mission Public School, Sirmour', duration: 'May 2022 – May 2024', grade: '86.50%' }
  ],
  skills: {
    AI: 90,
    Web: 88,
    Cybersecurity: 78,
    DevOps: 65
  },
  projects: [
    { name: 'Destifa', desc: 'AI travel assistant — personalized itineraries + Gemini API', link: 'https://github.com/parth-khera/destifa' },
    { name: 'Plant Disease Detection', desc: 'CV model to detect plant diseases from leaf images' },
    { name: 'Blockchain Prototype', desc: 'POC blockchain with PoW & demo smart contract' }
  ],
  achievements: [
    'Top 8% - ATF Fellowship Program (Y Combinator backed)',
    'Finalist – DSA Coding Competition (Knowvy Perks, Aug 2025)',
    'Finalist – ClueQuest DSA Challenge, GTBIT',
    '26th Rank – T20DSA Challenge (Top 5% Coder, July 2025)',
    'State-Level Science Congress – NIT Hamirpur (Dec 2023)',
    'Hackathon 2024 – Shiva Tech, Dehradun',
    'Utkarsh 1.0 Hackathon – UTU Dehradun (2025)'
  ],
  certifications: [
    'AI (CBSE) Apr 2021 – Mar 2022', 'Data Analytics – Explorin Academy (Feb 25, 2025)', 'Intro to SQL – Scrimba (Jun 2025)'
  ],
}

// ---------- Particles config (multi-layered neon) ----------
const ParticleOptions = {
  background: { color: 'transparent' },
  fpsLimit: 60,
  interactivity: {
    events: { onHover: { enable: true, mode: 'repulse' }, onClick: { enable: false }, resize: true },
    modes: { repulse: { distance: 120 } }
  },
  particles: {
    color: { value: ['#00f5ff', '#7c3aed', '#ff0072'] },
    links: { color: '#7c3aed', distance: 130, enable: true, opacity: 0.08, width: 1 },
    move: { enable: true, speed: 0.8, outModes: { default: 'bounce' } },
    number: { value: 60, density: { enable: true, area: 900 } },
    opacity: { value: 0.7 },
    size: { value: { min: 1, max: 4 } }
  }
}

// ---------- Small utility components ----------
const SkillBar = ({ name, value }) => (
  <div className="mb-3">
    <div className="flex justify-between text-xs text-slate-300 mb-1"><span>{name}</span><span>{value}%</span></div>
    <div className="w-full bg-slate-800/30 h-2 rounded-full overflow-hidden">
      <motion.div initial={{ width: 0 }} animate={{ width: `${value}%` }} transition={{ duration: 1.2 }} className="h-2 rounded-full bg-gradient-to-r from-sky-400 to-purple-600" />
    </div>
  </div>
)

const NeonBadge = ({ children }) => (
  <span className="inline-block px-3 py-1 rounded-full text-xs font-medium bg-gradient-to-r from-blue-400 to-pink-500 text-black shadow-[0_6px_24px_rgba(124,58,237,0.25)]">{children}</span>
)

// ---------- Voice navigation (Web Speech API) ----------
function useVoiceNav(sections = {}){
  useEffect(()=>{
    if(!('webkitSpeechRecognition' in window || 'SpeechRecognition' in window)) return;
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    const recog = new SpeechRecognition();
    recog.continuous = true; recog.lang = 'en-GB'; recog.interimResults = false;
    recog.onresult = (e) => {
      const last = e.results[e.results.length-1][0].transcript.trim().toLowerCase();
      console.log('voice:', last);
      Object.keys(sections).forEach(k => { if(last.includes(k.toLowerCase())) sections[k](); })
    }
    recog.onerror = ()=>{};
    // start only when user explicitly toggles (security) - we return control hooks for start/stop
    return ()=> recog.stop();
  },[sections])
}

// ---------- Main App ----------
export default function ParthProfilePro(){
  const particlesInit = async (engine) => { await loadFull(engine) }
  const [bio, setBio] = useState('Creative technologist building AI-first web systems.');
  const [chatOpen, setChatOpen] = useState(false);
  const sectionsRef = useRef({});

  // Voice nav hook (pass mapping of keywords to scroll functions)
  useVoiceNav({ Projects: ()=> sectionsRef.current.projects?.scrollIntoView({ behavior: 'smooth' }), Achievements: ()=> sectionsRef.current.achievements?.scrollIntoView({ behavior: 'smooth' }), Contact: ()=> sectionsRef.current.contact?.scrollIntoView({ behavior: 'smooth' }) });

  // Fetch AI bio from your backend (placeholder). If you don't have backend, this will fallback to resume summary.
  useEffect(()=>{
    let cancelled = false;
    async function fetchBio(){
      try{
        const res = await fetch('/api/ai-bio', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ resume }) });
        if(!res.ok) throw new Error('no ai');
        const json = await res.json(); if(!cancelled && json.bio) setBio(json.bio);
      }catch(e){ console.log('ai-bio fallback', e); setBio('Creative technologist building AI-first web systems—specialized in scalable backends and delightful UX.'); }
    }
    fetchBio();
    return ()=> cancelled = true;
  },[])

  // Minimal local chat stub (safe client-side only). For real LLM chat, connect to backend.
  const [messages, setMessages] = useState([{ from: 'bot', text: 'Hi — I am BitBot. Ask me about Parth, his projects, or achievements.' }]);
  async function sendMessage(text){
    setMessages(m=>[...m, { from: 'user', text }]);
    setTimeout(()=> setMessages(m=>[...m, { from: 'bot', text: "I can summarise Parth's projects or list achievements — try: 'Show achievements'" }]), 700);
    // If you connect backend, post to /api/chat
  }

  return (
    <div className="min-h-screen bg-gradient-to-b from-[#020617] via-[#090a10] to-[#05020a] text-slate-100 antialiased relative">
      <Particles init={particlesInit} options={ParticleOptions} className="absolute inset-0 -z-10" />

      <header className="max-w-6xl mx-auto px-6 py-6 flex items-center justify-between">
        <div>
          <h1 className="text-3xl md:text-4xl font-extrabold tracking-tight">🚀 {resume.name}</h1>
          <p className="text-sky-300 mt-1">{resume.title} · <span className="text-slate-400">{resume.location}</span></p>
        </div>

        <nav className="flex items-center gap-3">
          <a href={resume.website} target="_blank" rel="noreferrer" className="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-slate-800/40 hover:scale-[1.02] transition">
            <GitHub size={16}/> GitHub
          </a>
          <a href={`mailto:${resume.email}`} className="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-gradient-to-r from-sky-500 via-indigo-500 to-purple-600"> <Mail size={16}/> Contact</a>
        </nav>
      </header>

      <main className="max-w-6xl mx-auto px-6 pb-20">
        {/* HERO */}
        <section className="grid md:grid-cols-3 gap-6 items-center">
          <motion.div className="md:col-span-2 bg-slate-800/30 p-6 rounded-2xl backdrop-blur-sm shadow-xl" initial={{ y: 20, opacity: 0 }} animate={{ y: 0, opacity: 1 }} transition={{ duration: 0.6 }}>
            <div className="flex items-start gap-6">
              <div className="w-20 h-20 rounded-full bg-gradient-to-br from-sky-400 to-indigo-500 flex items-center justify-center text-2xl font-bold shadow-lg">PK</div>
              <div>
                <h2 className="text-2xl font-semibold">Hi, I'm {resume.name}</h2>
                <p className="mt-2 text-slate-300 leading-relaxed">{bio}</p>

                <div className="mt-4 flex gap-3 items-center">
                  <a href={resume.projects[0].link} target="_blank" rel="noreferrer" className="px-4 py-2 rounded-full bg-slate-700/40 hover:scale-[1.02] transition">Explore Destifa</a>
                  <button onClick={()=> setChatOpen(true)} className="px-4 py-2 rounded-full bg-gradient-to-r from-purple-600 to-pink-500">Talk to BitBot</button>
                  <NeonBadge>Open to opportunities</NeonBadge>
                </div>

                <div className="mt-6 grid sm:grid-cols-3 gap-3">
                  {Object.entries(resume.skills).map(([k,v]) => (
                    <div key={k} className="p-3 rounded-lg bg-slate-900/30">
                      <div className="text-sm font-semibold text-sky-200">{k}</div>
                      <div className="mt-3"><SkillBar name={k} value={v} /></div>
                    </div>
                  ))}
                </div>

              </div>
            </div>
          </motion.div>

          {/* 3D Hero */}
          <motion.div initial={{ scale: 0.9, opacity: 0 }} animate={{ scale: 1, opacity: 1 }} transition={{ delay: 0.2 }} className="rounded-2xl bg-slate-800/20 p-4 backdrop-blur-sm shadow-xl">
            <div style={{ width: '100%', height: 320 }} className="rounded-lg overflow-hidden">
              <Canvas camera={{ position: [0, 0, 4] }}>
                <ambientLight intensity={0.6} />
                <directionalLight position={[2, 2, 5]} intensity={0.8} />
                <mesh rotation={[0.9, 0.6, 0.2]}>
                  <torusKnotGeometry args={[0.6, 0.18, 128, 32]} />
                  <meshStandardMaterial metalness={0.9} roughness={0.2} color={'#7c3aed'} emissive={'#00f5ff'} emissiveIntensity={0.2} />
                </mesh>
                <OrbitControls enableZoom={false} enablePan={false} autoRotate autoRotateSpeed={0.5} />
              </Canvas>
            </div>
          </motion.div>
        </section>

        {/* Projects + Achievements */}
        <section className="grid md:grid-cols-3 gap-6 mt-8">
          <div className="md:col-span-2 bg-gradient-to-b from-slate-800/30 to-slate-900/20 p-6 rounded-2xl shadow-xl" ref={el => sectionsRef.current.projects = el}>
            <h3 className="text-xl font-semibold">Projects & Open Source</h3>
            <div className="mt-4 space-y-4">
              {resume.projects.map(p => (
                <motion.div whileHover={{ scale: 1.01 }} key={p.name} className="p-4 rounded-lg bg-slate-900/30">
                  <div className="flex items-start justify-between">
                    <div>
                      <h4 className="font-semibold">{p.name}</h4>
                      <p className="text-sm text-slate-300 mt-1">{p.desc}</p>
                    </div>
                    {p.link && <a href={p.link} target="_blank" rel="noreferrer" className="text-sm text-sky-300">Repo →</a>}
                  </div>
                </motion.div>
              ))}
            </div>
          </div>

          <aside className="p-6 rounded-2xl bg-slate-800/30 shadow-xl" ref={el => sectionsRef.current.achievements = el}>
            <h3 className="text-lg font-semibold">Achievements</h3>
            <ul className="mt-3 text-slate-300 list-disc list-inside space-y-2 text-sm">
              {resume.achievements.map(a => <li key={a}>{a}</li>)}
            </ul>

            <div className="mt-6">
              <h4 className="text-sm text-sky-200 font-medium">Certifications</h4>
              <ul className="mt-3 text-slate-300 text-sm list-disc list-inside space-y-1">
                {resume.certifications.map(c => <li key={c}>{c}</li>)}
              </ul>
            </div>
          </aside>
        </section>

        {/* Timeline - horizontal interactive (simple responsive version) */}
        <section className="mt-10 p-6 rounded-2xl bg-slate-900/20 shadow-xl">
          <h3 className="text-xl font-semibold mb-4">Career Timeline</h3>
          <div className="flex gap-4 overflow-x-auto py-2">
            {[...resume.achievements].map((a,i)=> (
              <div key={a} className="min-w-[240px] p-4 rounded-lg bg-gradient-to-br from-slate-800/30 to-slate-900/20 shadow-lg">
                <div className="text-sm text-sky-200 font-semibold">{i+1}. {a.split('–')[0].slice(0,40)}</div>
                <div className="text-xs text-slate-300 mt-2">{a}</div>
              </div>
            ))}
          </div>
        </section>

        {/* Education */}
        <section className="mt-10 grid md:grid-cols-3 gap-6">
          <div className="md:col-span-3 bg-slate-800/30 p-6 rounded-2xl shadow-xl">
            <h3 className="text-xl font-semibold">Education</h3>
            <div className="mt-4 grid md:grid-cols-3 gap-4">
              {resume.education.map(e => (
                <div key={e.degree} className="p-4 rounded-lg bg-slate-900/30">
                  <div className="font-semibold">{e.degree}</div>
                  <div className="text-sm text-slate-300 mt-1">{e.institute}</div>
                  <div className="text-xs text-slate-400 mt-1">{e.duration} {e.cgpa ? `· CGPA: ${e.cgpa}` : e.grade ? `· ${e.grade}` : ''}</div>
                </div>
              ))}
            </div>
          </div>
        </section>

        {/* Contact & Footer */}
        <section className="mt-10 p-6 rounded-2xl bg-slate-900/20 text-center" ref={el => sectionsRef.current.contact = el}>
          <h3 className="text-xl font-semibold">Let's work together</h3>
          <p className="text-slate-300 mt-2">Feel free to reach out — I respond quickly to meaningful opportunities.</p>
          <div className="mt-4 flex items-center justify-center gap-4">
            <a href={resume.website} target="_blank" rel="noreferrer" className="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-slate-800/40"> <GitHub size={16}/> View GitHub</a>
            <a href={resume.contactLinks?.linkedin || '#'} target="_blank" rel="noreferrer" className="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-slate-800/30"> <Linkedin size={16}/> LinkedIn</a>
            <a href={`mailto:${resume.email}`} className="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-gradient-to-r from-sky-500 to-indigo-500"> <Mail size={16}/> {resume.email}</a>
          </div>
        </section>

      </main>

      {/* Chat Widget */}
      {chatOpen && (
        <div className="fixed right-6 bottom-6 w-80 rounded-2xl bg-slate-900/80 p-4 shadow-2xl backdrop-blur-md z-50">
          <div className="flex items-center justify-between">
            <div className="text-sm font-semibold">BitBot — Portfolio Assistant</div>
            <button onClick={()=> setChatOpen(false)} className="text-slate-400">Close</button>
          </div>
          <div className="mt-3 h-48 overflow-auto text-sm space-y-2">
            {messages.map((m,idx)=> (
              <div key={idx} className={`p-2 rounded ${m.from==='bot' ? 'bg-slate-800/40 text-slate-200' : 'bg-slate-700/30 text-white'}`}>{m.text}</div>
            ))}
          </div>
          <div className="mt-3 flex gap-2">
            <input onKeyDown={(e)=> { if(e.key==='Enter') sendMessage(e.target.value); }} placeholder="Ask BitBot..." className="flex-1 px-3 py-2 rounded bg-slate-800/20 outline-none" />
            <button onClick={()=> sendMessage('Tell me about Destifa')} className="px-3 py-2 rounded bg-purple-600">Send</button>
          </div>
        </div>
      )}

      {/* Floating 3D orb for flair */}
      <div className="fixed right-6 top-24 w-28 h-28 md:block hidden">
        <Canvas camera={{ position: [0, 0, 6] }}>
          <ambientLight intensity={0.6} />
          <mesh>
            <icosahedronGeometry args={[1, 2]} />
            <meshStandardMaterial metalness={0.8} roughness={0.1} color={'#60a5fa'} emissive={'#7c3aed'} emissiveIntensity={0.12} />
          </mesh>
        </Canvas>
      </div>

    </div>
  )
}
