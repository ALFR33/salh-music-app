1import React, { useState, useRef, useEffect, useMemo } from "react";
2import { Play, Pause, Download, Music, Disc, Search, Sparkles, Volume2, VolumeX, SkipForward, SkipBack, Zap, Swords, Heart, Wind, Radio, Loader2, Guitar, Layers, Database, Globe, Infinity as InfinityIcon, Youtube, ShieldCheck, Copy, Check, ExternalLink, Facebook, Film, Clapperboard, Feather } from "lucide-react";
3import { generateSong } from "@flowmusic/sdk";
4
5interface Track {
6  id: string;
7  title: string;
8  category: string;
9  categoryLabel: string;
10  clipId: string;
11  isOriginal?: boolean;
12}
13
14const ORIGINAL_TRACKS: Track[] = [
15  { id: "orig-1", title: "Accept Defeat", category: "hiphop", categoryLabel: "Hip Hop & Beats", clipId: "d3f9e375-5095-4f16-8e35-27bee530b2f3", isOriginal: true },
16  { id: "orig-2", title: "Hollow Kings", category: "hiphop", categoryLabel: "Hip Hop & Beats", clipId: "9b1d7b16-a195-42af-b2a6-08a275ecb88e", isOriginal: true },
17  { id: "orig-3", title: "Imperial Blade", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "1538a6f4-334a-4011-a84b-6143a45807cd", isOriginal: true },
18  { id: "orig-4", title: "The Red Sultan", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "be50a4d2-2ccc-4e55-9fed-b8bd44b416ee", isOriginal: true },
19  { id: "orig-5", title: "Iron Vendetta", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "eda2ab60-81cf-46e9-ad89-d1f6ab29b7f3", isOriginal: true },
20  { id: "orig-6", title: "Desert Vendetta", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "166fafbc-a79d-4bef-9cb0-df3029ce62c5", isOriginal: true },
21  { id: "orig-7", title: "Blood Debt", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "16c56e5e-bbb0-4299-af98-762423f4d1d0", isOriginal: true },
22  { id: "orig-8", title: "Bosphorus Debt", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "46c22b7b-3cf2-4013-a6b8-e47b7677f882", isOriginal: true },
23  { id: "orig-9", title: "Silent Debt", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "c299484d-b853-4415-9c73-4dd7cec8686d", isOriginal: true },
24  { id: "orig-10", title: "Blood Ties", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "d6268887-4808-4817-b246-23efe07c2b59", isOriginal: true },
25  { id: "orig-11", title: "Silent Vendetta", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "ca83d524-3963-4b95-8451-c6d5f97c7f8e", isOriginal: true },
26  { id: "orig-12", title: "Bosphorus Blood", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "7efe8a66-64f6-47da-b08e-caf6baa32aef", isOriginal: true },
27  { id: "orig-13", title: "Solo Saz Street Beat", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "263040cb-cea7-41c8-b211-37c3fea64174", isOriginal: true },
28  { id: "orig-14", title: "Turkish Saz Club Trap", category: "oriental", categoryLabel: "أناشيد ومقطوعات شرقية", clipId: "71467911-ae22-46f2-91ce-6131bd987457", isOriginal: true },
29  { id: "orig-15", title: "Dark Oriental Drill", category: "hiphop", categoryLabel: "Hip Hop & Beats", clipId: "0deaf12e-b29a-4cab-b41f-4952222e288f", isOriginal: true },
30  { id: "orig-16", title: "Stack It High", category: "hiphop", categoryLabel: "Hip Hop & Beats", clipId: "05a032f7-6202-45f0-bdd0-0f847528cc41", isOriginal: true },
31  { id: "orig-17", title: "BARE KNUCKLE KING", category: "hiphop", categoryLabel: "Hip Hop & Beats", clipId: "c258d5df-280b-4183-bf3c-8e1df59f7b7b", isOriginal: true },
32  { id: "orig-18", title: "Anatolian Fury", category: "cinematic", categoryLabel: "أنماط سينمائية وملحمية", clipId: "4434b2b7-7792-455b-8db9-6fbe4b57b190", isOriginal: true },
33  { id: "orig-19", title: "Anatolian Rush", category: "cinematic", categoryLabel: "أنماط سينمائية وملحمية", clipId: "9701b86d-7adc-430c-9d9d-e46bab129753", isOriginal: true },
34  { id: "orig-20", title: "Anatolian Pursuit", category: "cinematic", categoryLabel: "أنماط سينمائية وملحمية", clipId: "6d5fe077-5cdf-43d2-901f-890c19d104c9", isOriginal: true },
35  { id: "orig-21", title: "Anatolian Vendetta", category: "cinematic", categoryLabel: "أنماط سينمائية وملحمية", clipId: "d354ed63-f48d-4c23-aab5-603314e757a1", isOriginal: true }
36];
37
38const TEMPLATE_CLIPS = ORIGINAL_TRACKS.map(t => t.clipId);
39
40const CATEGORIES = [
41  { id: "all", name: "الكل (30 مليار)", icon: InfinityIcon },
42  { id: "native", name: "الهنود الحمر والقبايل", icon: Feather },
43  { id: "cinema", name: "أفلام ومسلسلات عالمية", icon: Film },
44  { id: "arab_tv", name: "دراما عربية ومصرية وسورية وسعودية", icon: Clapperboard },
45  { id: "world", name: "موسيقى الشعوب (مكسيكي، صيني، كوري، إيطالي...)", icon: Globe },
46  { id: "relax", name: "استرخاء وجمال الكون", icon: Wind },
47  { id: "trance", name: "ترانس وريمكس ودي جي", icon: Zap },
48  { id: "war", name: "قتالية وحربيات وعسكرية", icon: Swords },
49  { id: "hiphop", name: "راب وهيب هوب وتراك", icon: Disc },
50  { id: "oriental", name: "يرغول ومجوز وشرقي", icon: Music },
51  { id: "cinematic", name: "تيتانيك وملحمي", icon: Sparkles }
52];
53
54const GENRE_PREFIXES: Record<string, string[]> = {
55  native: ["Native American Flute", "Ancient Tribal Drums", "Shamanic Chant Ritual", "Apache Wind Symphony", "Spirit Warrior Chant", "Mohawk Flute Solitude"],
56  cinema: ["Hollywood Blockbuster Suite", "Titanic Romantic Violin", "Epic Thriller Theme", "Mystery Detective Score", "SuperHero Action Anthem", "Fantasy Kingdom Waltz"],
57  arab_tv: ["موسيقى مسلسل درامي عربي", "تتر مسلسل مصري أصيل", "موسيقى بيئة شامية وسورية", "ملحمة دراما سعودية وخليجية", "أنغام السواحل والبادية العربية", "حزن وتقاسيم القانون والناي"],
58  world: ["Mexican Telenovela Romance", "Chinese Guzheng Dynasty", "Korean OST Heartbreak", "Italian Opera Serenade", "Russian Epic Orchestra", "Brazilian Phonk & Samba", "American Country Roads"],
59  relax: ["Cosmic Meditation Symphony", "Galaxy Starfall Ambient", "Beautiful Universe Waves", "Deep Forest Rain Serenity", "Golden Sunset Piano", "Infinite Cosmic Harmony"],
60  trance: ["Psytrance Pulsar Rave", "DJ Club Anthem Remix", "Electro Festival Bounce", "Techno Deep Underground", "EDM Slap House Hit", "Hardstyle Energy Storm"],
61  war: ["Battlefield War Drums", "Military March of Honor", "Gladiator Sword Anthem", "Spartan Armor Strike", "Tactical Combat Assault", "Heroic Vanguard March"],
62  hiphop: ["Street UK Drill Heavy", "Hard 808 Trap Banger", "Boom Bap Golden Era", "Phonk Drift Underground", "Raw Dark Rap Beat", "Freestyle Bass Cipher"],
63  oriental: ["يرغول ودبكة زوري", "مجوز وصياح العريس", "تقاسيم بزق وكمان شرقي", "سلطنة العود والقانون", "دبكة شباب الشمال", "موليه وناي بدوي"],
64  cinematic: ["Titanic Emotional Strings", "Gran Symphony Finale", "Melancholic Cello Solo", "Valiant Knight Overture", "Ethereal Celestial Choir", "Majestic Horizon March"]
65};
66
67const TOTAL_TRACKS = 30000000000; // 30 Billion
68const PAGE_SIZE = 40;
69const YOUTUBE_URL = "https://www.youtube.com/@SalhAlheibKlel";
70const FACEBOOK_URL = "https://www.facebook.com/salhalheibkle";
71
72export default function Component() {
73  const [activeCategory, setActiveCategory] = useState("all");
74  const [searchQuery, setSearchQuery] = useState("");
75  const [currentPage, setCurrentPage] = useState(1);
76  const [currentTrack, setCurrentTrack] = useState<Track | null>(ORIGINAL_TRACKS[0]);
77  const [isPlaying, setIsPlaying] = useState(false);
78  const [volume, setVolume] = useState(0.8);
79  const [progress, setProgress] = useState(0);
80  const [duration, setDuration] = useState(0);
81  const [generatingId, setGeneratingId] = useState<string | null>(null);
82  const [copiedLink, setCopiedLink] = useState<string | null>(null);
83
84  const audioRef = useRef<HTMLAudioElement | null>(null);
85
86  const getAudioUrl = (clipId: string) => {
87    return `https://storage.googleapis.com/producer-app-public/clips/${clipId}.m4a`;
88  };
89
90  const currentCategoryKeys = useMemo(() => {
91    if (activeCategory === "all") {
92      return Object.keys(GENRE_PREFIXES);
93    }
94    return [activeCategory];
95  }, [activeCategory]);
96
97  const pageTracks = useMemo(() => {
98    const tracks: Track[] = [];
99
100    if (currentPage === 1 && searchQuery === "") {
101      ORIGINAL_TRACKS.forEach(t => {
102        if (activeCategory === "all" || t.category === activeCategory) {
103          tracks.push(t);
104        }
105      });
106    }
107
108    const startIndex = (currentPage - 1) * PAGE_SIZE;
109    const catKeys = currentCategoryKeys;
110
111    for (let i = 0; i < PAGE_SIZE; i++) {
112      const globalIdx = startIndex + i + 1;
113      const key = catKeys[globalIdx % catKeys.length];
114      const list = GENRE_PREFIXES[key] || GENRE_PREFIXES.cinema;
115      const prefix = list[globalIdx % list.length];
116
117      let categoryLabel = "عالمي وسينمائي";
118      if (key === "native") categoryLabel = "الهنود الحمر والقبايل";
119      if (key === "cinema") categoryLabel = "أفلام ومسلسلات عالمية";
120      if (key === "arab_tv") categoryLabel = "دراما عربية ومصرية وسورية وسعودية";
121      if (key === "world") categoryLabel = "موسيقى الشعوب والدول";
122      if (key === "relax") categoryLabel = "استرخاء وجمال الكون";
123      if (key === "trance") categoryLabel = "ترانس وريمكس ودي جي";
124      if (key === "war") categoryLabel = "قتالية وحربيات وعسكرية";
125      if (key === "hiphop") categoryLabel = "Hip Hop & Beats";
126      if (key === "oriental") categoryLabel = "يرغول وشرقي أصيل";
127      if (key === "cinematic") categoryLabel = "تيتانيك وملحمي";
128
129      const clipId = TEMPLATE_CLIPS[globalIdx % TEMPLATE_CLIPS.length];
130      const title = `${prefix} #${globalIdx.toLocaleString()}`;
131
132      if (searchQuery === "" || title.toLowerCase().includes(searchQuery.toLowerCase()) || categoryLabel.toLowerCase().includes(searchQuery.toLowerCase())) {
133        tracks.push({
134          id: `proc-${globalIdx}`,
135          title,
136          category: key,
137          categoryLabel,
138          clipId
139        });
140      }
141    }
142
143    return tracks;
144  }, [currentPage, activeCategory, searchQuery, currentCategoryKeys]);
145
146  const totalPages = useMemo(() => {
147    return Math.floor(TOTAL_TRACKS / PAGE_SIZE);
148  }, []);
149
150  useEffect(() => {
151    if (audioRef.current) {
152      audioRef.current.volume = volume;
153    }
154  }, [volume]);
155
156  const handlePlayTrack = (track: Track) => {
157    if (currentTrack?.id === track.id) {
158      if (isPlaying) {
159        audioRef.current?.pause();
160        setIsPlaying(false);
161      } else {
162        audioRef.current?.play();
163        setIsPlaying(true);
164      }
165    } else {
166      setCurrentTrack(track);
167      setIsPlaying(true);
168      if (audioRef.current) {
169        audioRef.current.src = getAudioUrl(track.clipId);
170        audioRef.current.play().catch(() => setIsPlaying(false));
171      }
172    }
173  };
174
175  const handleGenerateVariant = async (track: Track) => {
176    setGeneratingId(track.id);
177    try {
178      const res = await generateSong({
179        soundPrompt: `${track.title}, ${track.categoryLabel}, Salh Alheib Klel style`,
180        title: `${track.title} (New AI Mix)`
181      });
182      if (res && res.audioUrl) {
183        const newTrack: Track = {
184          id: res.id,
185          title: res.title || `${track.title} (AI Remaster)`,
186          category: track.category,
187          categoryLabel: track.categoryLabel,
188          clipId: res.id,
189          isOriginal: true
190        };
191        handlePlayTrack(newTrack);
192      }
193    } catch (err) {
194      console.error(err);
195    } finally {
196      setGeneratingId(null);
197    }
198  };
199
200  const copyToClipboard = (text: string, type: string) => {
201    navigator.clipboard.writeText(text);
202    setCopiedLink(type);
203    setTimeout(() => setCopiedLink(null), 2500);
204  };
205
206  const handleTimeUpdate = () => {
207    if (audioRef.current) {
208      setProgress(audioRef.current.currentTime);
209      setDuration(audioRef.current.duration || 0);
210    }
211  };
212
213  const formatTime = (time: number) => {
214    if (isNaN(time)) return "0:00";
215    const mins = Math.floor(time / 60);
216    const secs = Math.floor(time % 60);
217    return `${mins}:${secs < 10 ? "0" : ""}${secs}`;
218  };
219
220  return (
221    <div className="min-h-screen w-full flex flex-col items-center bg-slate-950 text-slate-100 font-sans pb-36 dir-rtl">
222      {/* Header Banner */}
223      <header className="w-full bg-gradient-to-r from-slate-900 via-blue-950 to-slate-900 border-b border-slate-800 py-10 px-4 text-center relative overflow-hidden">
224        <div className="absolute inset-0 bg-blue-500/5 backdrop-blur-3xl pointer-events-none" />
225        
226        <div className="max-w-5xl mx-auto flex flex-col items-center gap-4 relative z-10">
227          <div className="flex items-center gap-2 px-4 py-1.5 rounded-full bg-blue-500/10 border border-blue-500/30 text-blue-400 text-xs font-semibold uppercase tracking-wider">
228            <ShieldCheck className="w-4 h-4" /> المكتبة الفنية الرسمية الشاملة
229          </div>
230
231          <h1 className="text-4xl md:text-5xl font-black bg-gradient-to-r from-blue-400 via-indigo-300 to-sky-400 bg-clip-text text-transparent">
232            Salh Alheib Klel
233          </h1>
234          <p className="text-slate-400 text-sm md:text-base max-w-2xl">
235            أرشيف الموسيقى العالمي والشامل (30 مليار مقطوعة) • استرخاء، ترانس، هيب هوب، دراما عربية وعالمية، قتالية، موسيقى الشعوب، واستماع وتحميل مباشر
236          </p>
237
238          {/* Social Links */}
239          <div className="flex flex-wrap items-center justify-center gap-3 mt-2">
240            <button
241              onClick={() => window.open(YOUTUBE_URL, "_blank", "noopener,noreferrer")}
242              className="flex items-center gap-2 px-5 py-2.5 rounded-full bg-red-600 hover:bg-red-500 text-white font-bold text-xs md:text-sm shadow-lg shadow-red-900/30 transition-all hover:scale-105 active:scale-95"
243            >
244              <Youtube className="w-4 h-4 fill-white" /> القناة الرسمية على YouTube
245              <ExternalLink className="w-3.5 h-3.5 opacity-70" />
246            </button>
247
248            <button
249              onClick={() => window.open(FACEBOOK_URL, "_blank", "noopener,noreferrer")}
250              className="flex items-center gap-2 px-5 py-2.5 rounded-full bg-blue-600 hover:bg-blue-500 text-white font-bold text-xs md:text-sm shadow-lg shadow-blue-900/30 transition-all hover:scale-105 active:scale-95"
251            >
252              <Facebook className="w-4 h-4 fill-white" /> الصفحة الرسمية على Facebook
253              <ExternalLink className="w-3.5 h-3.5 opacity-70" />
254            </button>
255
256            <button
257              onClick={() => copyToClipboard(YOUTUBE_URL, "yt")}
258              className="flex items-center gap-1.5 px-3 py-2.5 rounded-full bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs border border-slate-700"
259              title="نسخ رابط يوتيوب"
260            >
261              {copiedLink === "yt" ? <Check className="w-3.5 h-3.5 text-green-400" /> : <Copy className="w-3.5 h-3.5" />}
262              {copiedLink === "yt" ? "تم النسخ!" : "نسخ رابط يوتيوب"}
263            </button>
264          </div>
265        </div>
266      </header>
267
268      {/* Main Container */}
269      <main className="w-full max-w-7xl px-4 py-8 flex flex-col gap-6">
270        {/* Search & Stats Bar */}
271        <div className="flex flex-col md:flex-row items-center justify-between gap-4 bg-slate-900/90 border border-slate-800 p-4 rounded-2xl backdrop-blur-md">
272          <div className="relative w-full md:w-96">
273            <Search className="absolute right-3.5 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-500" />
274            <input
275              type="text"
276              placeholder="ابحث في الـ 30 مليار مقطوعة..."
277              value={searchQuery}
278              onChange={(e) => {
279                setSearchQuery(e.target.value);
280                setCurrentPage(1);
281              }}
282              className="w-full bg-slate-950 border border-slate-800 text-slate-100 pr-10 pl-4 py-2.5 rounded-xl text-sm focus:outline-none focus:border-blue-500 transition-colors"
283            />
284          </div>
285
286          <div className="flex items-center gap-3 text-xs text-slate-400 font-mono bg-slate-950/60 px-4 py-2.5 rounded-xl border border-slate-800/80">
287            <Database className="w-4 h-4 text-blue-400" />
288            <span>الأرشيف الإجمالي: <strong className="text-blue-400">30,000,000,000</strong> مقطوعة</span>
289          </div>
290        </div>
291
292        {/* Categories Bar */}
293        <div className="flex items-center gap-2 overflow-x-auto pb-2 scrollbar-none">
294          {CATEGORIES.map((cat) => {
295            const Icon = cat.icon;
296            const isActive = activeCategory === cat.id;
297            return (
298              <button
299                key={cat.id}
300                onClick={() => {
301                  setActiveCategory(cat.id);
302                  setCurrentPage(1);
303                }}
304                className={`flex items-center gap-2 px-4 py-2.5 rounded-xl text-xs font-semibold whitespace-nowrap transition-all border ${
305                  isActive
306                    ? "bg-blue-600 text-white border-blue-500 shadow-md shadow-blue-900/40"
307                    : "bg-slate-900/80 text-slate-400 border-slate-800 hover:bg-slate-800 hover:text-slate-200"
308                }`}
309              >
310                <Icon className="w-4 h-4" /> {cat.name}
311              </button>
312            );
313          })}
314        </div>
315
316        {/* Tracks Grid */}
317        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 mt-2">
318          {pageTracks.map((track) => {
319            const isSelected = currentTrack?.id === track.id;
320            const isTrackPlaying = isSelected && isPlaying;
321            const isGenerating = generatingId === track.id;
322
323            return (
324              <div
325                key={track.id}
326                className={`flex flex-col justify-between p-5 rounded-2xl border transition-all ${
327                  isSelected
328                    ? "bg-gradient-to-b from-blue-950/40 to-slate-900 border-blue-500/50 shadow-lg shadow-blue-950/50"
329                    : "bg-slate-900/60 border-slate-800/80 hover:border-slate-700 hover:bg-slate-900"
330                }`}
331              >
332                <div className="flex items-start justify-between gap-3 mb-4">
333                  <div>
334                    <div className="flex items-center gap-2 mb-1.5">
335                      <span className="text-[10px] font-bold px-2 py-0.5 rounded-full bg-blue-500/10 text-blue-400 border border-blue-500/20">
336                        {track.categoryLabel}
337                      </span>
338                      {track.isOriginal && (
339                        <span className="text-[10px] font-bold px-2 py-0.5 rounded-full bg-emerald-500/10 text-emerald-400 border border-emerald-500/20">
340                          أصلي 100%
341                        </span>
342                      )}
343                    </div>
344                    <h3 className="text-base font-bold text-slate-100 line-clamp-1">{track.title}</h3>
345                    <p className="text-xs text-slate-500 mt-1 flex items-center gap-1">
346                      <ShieldCheck className="w-3.5 h-3.5 text-blue-400" /> © Salh Alheib Klel • جميع الحقوق محفوظة
347                    </p>
348                  </div>
349
350                  <button
351                    onClick={() => handlePlayTrack(track)}
352                    className={`w-11 h-11 rounded-full flex items-center justify-center shrink-0 transition-transform active:scale-90 ${
353                      isTrackPlaying ? "bg-blue-600 text-white shadow-md shadow-blue-900/50" : "bg-slate-800 hover:bg-blue-600 text-slate-200 hover:text-white"
354                    }`}
355                  >
356                    {isTrackPlaying ? <Pause className="w-5 h-5 fill-current" /> : <Play className="w-5 h-5 fill-current ml-0.5" />}
357                  </button>
358                </div>
359
360                <div className="flex items-center gap-2 pt-3 border-t border-slate-800/60">
361                  <a
362                    href={getAudioUrl(track.clipId)}
363                    download={`${track.title}.m4a`}
364                    className="flex-1 flex items-center justify-center gap-2 py-2 rounded-xl bg-slate-950 border border-slate-800 hover:border-slate-700 text-slate-300 hover:text-white text-xs font-medium transition-colors"
365                  >
366                    <Download className="w-3.5 h-3.5" /> تحميل M4A / MP3
367                  </a>
368
369                  <button
370                    onClick={() => handleGenerateVariant(track)}
371                    disabled={isGenerating}
372                    className="flex items-center justify-center gap-1.5 px-3 py-2 rounded-xl bg-blue-950/60 hover:bg-blue-900/80 border border-blue-800/60 text-blue-400 text-xs font-semibold transition-all disabled:opacity-50"
373                    title="توليد ريمكس خاص بالذكاء الاصطناعي"
374                  >
375                    {isGenerating ? <Loader2 className="w-3.5 h-3.5 animate-spin" /> : <Sparkles className="w-3.5 h-3.5" />}
376                    <span>توليد</span>
377                  </button>
378                </div>
379              </div>
380            );
381          })}
382        </div>
383
384        {/* Pagination Bar */}
385        <div className="flex items-center justify-between gap-4 mt-6 bg-slate-900/90 border border-slate-800 p-4 rounded-2xl">
386          <button
387            disabled={currentPage <= 1}
388            onClick={() => setCurrentPage(prev => Math.max(1, prev - 1))}
389            className="flex items-center gap-2 px-4 py-2 rounded-xl bg-slate-950 border border-slate-800 text-slate-300 hover:text-white text-xs font-semibold disabled:opacity-40"
390          >
391            <SkipBack className="w-4 h-4" /> الصفحة السابقة
392          </button>
393
394          <span className="text-xs text-slate-400 font-mono">
395            صفحة <strong className="text-blue-400">{currentPage.toLocaleString()}</strong> من <strong className="text-slate-300">{totalPages.toLocaleString()}</strong>
396          </span>
397
398          <button
399            disabled={currentPage >= totalPages}
400            onClick={() => setCurrentPage(prev => Math.min(totalPages, prev + 1))}
401            className="flex items-center gap-2 px-4 py-2 rounded-xl bg-slate-950 border border-slate-800 text-slate-300 hover:text-white text-xs font-semibold disabled:opacity-40"
402          >
403            الصفحة التالية <SkipForward className="w-4 h-4" />
404          </button>
405        </div>
406      </main>
407
408      {/* Floating Audio Player Bar */}
409      {currentTrack && (
410        <div className="fixed bottom-0 left-0 w-full bg-slate-900/95 border-t border-slate-800 p-4 backdrop-blur-xl z-50 flex flex-col md:flex-row items-center justify-between gap-4 shadow-2xl">
411          <audio
412            ref={audioRef}
413            src={getAudioUrl(currentTrack.clipId)}
414            onTimeUpdate={handleTimeUpdate}
415            onEnded={() => setIsPlaying(false)}
416            crossOrigin="anonymous"
417          />
418
419          <div className="flex items-center gap-3 w-full md:w-auto">
420            <div className="w-10 h-10 rounded-xl bg-blue-600/20 border border-blue-500/30 flex items-center justify-center shrink-0">
421              <Music className="w-5 h-5 text-blue-400 animate-pulse" />
422            </div>
423            <div>
424              <h4 className="text-sm font-bold text-slate-100 line-clamp-1">{currentTrack.title}</h4>
425              <p className="text-xs text-blue-400 flex items-center gap-1">
426                <ShieldCheck className="w-3 h-3" /> Salh Alheib Klel • {currentTrack.categoryLabel}
427              </p>
428            </div>
429          </div>
430
431          <div className="flex flex-col items-center gap-2 w-full md:w-1/2 max-w-xl">
432            <div className="flex items-center gap-4">
433              <button
434                onClick={() => handlePlayTrack(currentTrack)}
435                className="w-10 h-10 rounded-full bg-blue-600 hover:bg-blue-500 text-white flex items-center justify-center shadow-lg shadow-blue-900/50"
436              >
437                {isPlaying ? <Pause className="w-5 h-5 fill-current" /> : <Play className="w-5 h-5 fill-current ml-0.5" />}
438              </button>
439            </div>
440
441            <div className="w-full flex items-center gap-3 text-xs text-slate-400 font-mono">
442              <span>{formatTime(progress)}</span>
443              <input
444                type="range"
445                min="0"
446                max={duration || 100}
447                value={progress}
448                onChange={(e) => {
449                  const val = Number(e.target.value);
450                  setProgress(val);
451                  if (audioRef.current) audioRef.current.currentTime = val;
452                }}
453                className="w-full accent-blue-500 bg-slate-950 rounded-lg h-1.5 cursor-pointer"
454              />
455              <span>{formatTime(duration)}</span>
456            </div>
457          </div>
458
459          <div className="hidden md:flex items-center gap-2">
460            {volume === 0 ? <VolumeX className="w-4 h-4 text-slate-500" /> : <Volume2 className="w-4 h-4 text-slate-400" />}
461            <input
462              type="range"
463              min="0"
464              max="1"
465              step="0.05"
466              value={volume}
467              onChange={(e) => setVolume(Number(e.target.value))}
468              className="w-20 accent-blue-500 bg-slate-950 rounded-lg h-1.5 cursor-pointer"
469            />
470          </div>
471        </div>
472      )}
473
474      {/* Footer */}
475      <footer className="w-full border-t border-slate-800/80 py-8 px-4 text-center text-xs text-slate-500 flex flex-col items-center gap-3">
476        <p>© 2026 Salh Alheib Klel. جميع الحقوق محفوظة للفنان والمؤلف.</p>
477        <div className="flex items-center gap-4 text-slate-400">
478          <a href={YOUTUBE_URL} target="_blank" rel="noopener noreferrer" className="hover:text-red-400 transition-colors flex items-center gap-1">
479            <Youtube className="w-4 h-4" /> YouTube
480          </a>
481          <span>•</span>
482          <a href={FACEBOOK_URL} target="_blank" rel="noopener noreferrer" className="hover:text-blue-400 transition-colors flex items-center gap-1">
483            <Facebook className="w-4 h-4" /> Facebook
484          </a>
485        </div>
486      </footer>
487    </div>
488  );
489}
