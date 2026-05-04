eState, useCallback } from 'react';
import ParticleBackground from './components/ParticleBackground';
import Header from './components/Header';
import QueryInput from './components/QueryInput';
import ModelCard from './components/ModelCard';
import SynthesisPanel from './components/SynthesisPanel';
import StatsBar from './components/StatsBar';
import { AI_MODELS } from './data/models';

type AppState = 'idle' | 'processing' | 'synthesizing' | 'complete';

export default function App() {
  const [appState, setAppState] = useState<AppState>('idle');
  const [currentQuery, setCurrentQuery] = useState('');
  const [completedModels, setCompletedModels] = useState<Set<string>>(new Set());
  const [showSynthesis, setShowSynthesis] = useState(false);
  const [hasQueried, setHasQueried] = useState(false);
  const [confidenceMap, setConfidenceMap] = useState<Record<string, number>>({});

  const handleSubmit = useCallback((query: string) => {
    setCurrentQuery(query);
    setAppState('processing');
    setCompletedModels(new Set());
    setShowSynthesis(false);
    setHasQueried(true);
    setConfidenceMap({});

    // Simulate each model completing at different times
    const confidences: Record<string, number> = {
      gpt4o: 0.92, claude35: 0.95, gemini15: 0.89,
      llama3: 0.87, mistral: 0.91, grok2: 0.85,
      deepseek: 0.93, perplexity: 0.88,
    };

    AI_MODELS.forEach((model) => {
      const totalTime = model.processingTime + (Math.random() * 400 - 200);
      setTimeout(() => {
        setCompletedModels((prev) => {
          const next = new Set(prev);
          next.add(model.id);
          return next;
        });
        setConfidenceMap((prev) => ({ ...prev, [model.id]: confidences[model.id] }));
      }, totalTime);
    });

    // After all models complete, start synthesis
    const maxTime = Math.max(...AI_MODELS.map((m) => m.processingTime)) + 800;
    setTimeout(() => {
      setAppState('synthesizing');
      setTimeout(() => {
        setAppState('complete');
        setShowSynthesis(true);
      }, 600);
    }, maxTime);
  }, []);

  const handleReset = useCallback(() => {
    setAppState('idle');
    setCurrentQuery('');
    setCompletedModels(new Set());
    setShowSynthesis(false);
    setHasQueried(false);
    setConfidenceMap({});
  }, []);

  const isProcessing = appState === 'processing' || appState === 'synthesizing';
  const isComplete = appState === 'complete';

  return (
    <div className="min-h-screen bg-gray-950 text-white font-['Inter'] overflow-x-hidden">
      {/* Particle background */}
      <ParticleBackground />

      {/* Radial glow effects */}
      <div className="fixed inset-0 pointer-events-none z-0 overflow-hidden">
        <div
          className="absolute -top-40 -left-40 w-96 h-96 rounded-full opacity-10"
          style={{ background: 'radial-gradient(circle, #8b5cf6 0%, transparent 70%)' }}
        />
        <div
          className="absolute -bottom-40 -right-40 w-96 h-96 rounded-full opacity-10"
          style={{ background: 'radial-gradient(circle, #06b6d4 0%, transparent 70%)' }}
        />
        <div
          className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[800px] h-[800px] rounded-full opacity-5"
          style={{ background: 'radial-gradient(circle, #7c3aed 0%, transparent 70%)' }}
        />
      </div>

      {/* Header */}
      <Header onReset={handleReset} />

      {/* Main content */}
      <main className="relative z-10 max-w-7xl mx-auto px-4 sm:px-6 py-8">

        {/* Hero section — shown when idle or as header when queried */}
        <div className={`text-center transition-all duration-700 ${hasQueried ? 'mb-6' : 'mb-10 mt-8'}`}>
          {!hasQueried && (
            <>
              {/* Badge */}
              <div className="inline-flex items-center gap-2 bg-violet-500/10 border border-violet-500/30 text-violet-300 text-xs font-medium px-4 py-1.5 rounded-full mb-6">
                <div className="w-1.5 h-1.5 rounded-full bg-violet-400 animate-pulse" />
                8 Modeli AI • Analiza Równoległa • Synteza Synergiczna
              </div>

              {/* Main heading */}
              <h1 className="text-5xl sm:text-6xl lg:text-7xl font-black mb-4 tracking-tight leading-none">
                <span className="text-white">Jeden interfejs.</span>
                <br />
                <span className="bg-gradient-to-r from-violet-400 via-purple-400 to-cyan-400 bg-clip-text text-transparent">
                  Wszystkie AI.
                </span>
              </h1>

              {/* Subtitle */}
              <p className="text-gray-400 text-lg max-w-2xl mx-auto mb-3 leading-relaxed">
                Wpisz zapytanie raz. SynapseAI aktywuje wszystkie zintegrowane modele AI
                <span className="text-violet-400 font-medium"> równolegle</span>, syntetyzując ich odpowiedzi
                w jedną, optymalną i wielowymiarową odpowiedź.
              </p>

              {/* Model icons row */}
              <div className="flex items-center justify-center gap-2 mb-10 flex-wrap">
                {AI_MODELS.map((model) => (
                  <div
                    key={model.id}
                    className={`flex items-center gap-1.5 bg-gray-900/70 border ${model.borderColor} rounded-full px-3 py-1`}
                  >
                    <span className="text-sm">{model.icon}</span>
                    <span className="text-xs text-gray-400 font-medium">{model.shortName}</span>
                  </div>
                ))}
              </div>
            </>
          )}

          {/* Query display when processing/complete */}
          {hasQueried && currentQuery && (
            <div className="flex items-center justify-center gap-3 mb-2">
              <div className="w-2 h-2 rounded-full bg-violet-500 animate-pulse" />
              <p className="text-gray-400 text-sm">
                Zapytanie: <span className="text-white font-medium">"{currentQuery.length > 80 ? currentQuery.slice(0, 80) + '...' : currentQuery}"</span>
              </p>
              <button onClick={handleReset} className="text-gray-600 hover:text-gray-400 text-xs border border-gray-800 hover:border-gray-700 px-2 py-1 rounded-lg transition-all">
                Nowe zapytanie
              </button>
            </div>
          )}
        </div>

        {/* Query input */}
        {!hasQueried && (
          <QueryInput
            onSubmit={handleSubmit}
            isProcessing={isProcessing}
            disabled={isComplete}
          />
        )}

        {/* Stats bar */}
        <StatsBar
          completedModels={completedModels.size}
          totalModels={AI_MODELS.length}
          isProcessing={isProcessing}
          isSynthesisReady={isComplete}
        />

        {/* Processing view */}
        {hasQueried && (
          <div className="mt-8">
            {/* Section header */}
            <div className="flex items-center justify-between mb-4">
              <div className="flex items-center gap-2">
                <h2 className="text-gray-300 font-semibold text-sm">
                  Modele AI — Analiza równoległa
                </h2>
                {isProcessing && (
                  <span className="text-xs text-violet-400 bg-violet-500/10 border border-violet-500/20 px-2 py-0.5 rounded-full animate-pulse">
                    LIVE
                  </span>
                )}
              </div>
              {isComplete && (
                <span className="text-xs text-emerald-400 flex items-center gap-1">
                  <svg className="w-3 h-3" viewBox="0 0 24 24" fill="none">
                    <path d="M20 6L9 17L4 12" stroke="currentColor" strokeWidth="2.5" strokeLinecap="round" strokeLinejoin="round"/>
                  </svg>
                  Wszystkie modele zakończone
                </span>
              )}
            </div>

            {/* Model cards grid */}
            <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-3">
              {AI_MODELS.map((model, index) => (
                <ModelCard
                  key={model.id}
                  model={model}
                  isProcessing={isProcessing || isComplete}
                  isComplete={completedModels.has(model.id)}
                  confidence={confidenceMap[model.id]}
                  delay={index * 80}
                />
              ))}
            </div>

            {/* Synthesis arrow */}
            {(appState === 'synthesizing' || appState === 'complete') && (
              <div className="flex flex-col items-center my-6">
                <div className="flex items-center gap-3">
                  <div className="h-px w-20 bg-gradient-to-r from-transparent to-violet-500/60" />
                  <div className={`flex items-center gap-2 bg-gray-900 border border-violet-500/40 rounded-full px-4 py-2 ${appState === 'synthesizing' ? 'animate-pulse' : ''}`}>
                    <div className="w-2 h-2 rounded-full bg-gradient-to-r from-violet-500 to-cyan-500" />
                    <span className="text-xs font-semibold text-violet-300 tracking-widest uppercase">
                      {appState === 'synthesizing' ? 'Synteza...' : 'Synteza Gotowa'}
                    </span>
                    <div className="w-2 h-2 rounded-full bg-gradient-to-r from-cyan-500 to-violet-500" />
                  </div>
                  <div className="h-px w-20 bg-gradient-to-l from-transparent to-cyan-500/60" />
                </div>
                <svg className="w-5 h-5 text-violet-500 mt-2" viewBox="0 0 24 24" fill="none">
                  <path d="M12 5V19M12 19L5 12M12 19L19 12" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"/>
                </svg>
              </div>
            )}

            {/* Synthesis result */}
            <SynthesisPanel
              isVisible={showSynthesis}
              query={currentQuery}
              modelsCount={AI_MODELS.length}
            />

            {/* New query button after complete */}
            {isComplete && (
              <div className="mt-6 text-center">
                <button
                  onClick={handleReset}
                  className="inline-flex items-center gap-2 bg-gradient-to-r from-violet-600 to-purple-600 hover:from-violet-500 hover:to-purple-500 text-white px-8 py-3 rounded-xl font-semibold text-sm transition-all shadow-lg shadow-violet-900/40 hover:shadow-violet-900/60 hover:scale-105 active:scale-95"
                >
                  <svg className="w-4 h-4" viewBox="0 0 24 24" fill="none">
                    <path d="M12 5V19M5 12H19" stroke="currentColor" strokeWidth="2" strokeLinecap="round"/>
                  </svg>
                  Nowe zapytanie do wszystkich AI
                </button>
              </div>
            )}
          </div>
        )}

        {/* Feature cards — only on idle */}
        {!hasQueried && (
          <div className="mt-16 grid grid-cols-1 md:grid-cols-3 gap-4">
            {[
              {
                icon: '⚡',
                title: 'Analiza Równoległa',
                desc: 'Wszystkie modele AI pracują jednocześnie — nie czekasz na każdy z osobna. Maksymalna efektywność.',
                color: 'from-violet-500/10 to-violet-600/5',
                border: 'border-violet-500/20',
              },
              {
                icon: '🔮',
                title: 'Synteza Synergiczna',
                desc: 'Odpowiedzi wszystkich modeli są scalane w jedną, kompletną syntezę o jakości przewyższającej każdy model z osobna.',
                color: 'from-cyan-500/10 to-cyan-600/5',
                border: 'border-cyan-500/20',
              },
              {
                icon: '🎯',
                title: 'Jeden Interfejs',
                desc: 'Koniec z przełączaniem między ChatGPT, Claude, Gemini i innymi. Wszystko dostępne z jednego miejsca.',
                color: 'from-emerald-500/10 to-emerald-600/5',
                border: 'border-emerald-500/20',
              },
            ].map((f) => (
              <div
                key={f.title}
                className={`relative bg-gradient-to-br ${f.color} border ${f.border} rounded-2xl p-6 backdrop-blur-sm overflow-hidden`}
              >
                <div className="text-3xl mb-3">{f.icon}</div>
                <h3 className="text-white font-bold text-base mb-2">{f.title}</h3>
                <p className="text-gray-500 text-sm leading-relaxed">{f.desc}</p>
              </div>
            ))}
          </div>
        )}
      </main>

      {/* Footer */}
      <footer className="relative z-10 border-t border-gray-800/40 mt-16 py-6 text-center">
        <p className="text-gray-700 text-xs">
          SynapseAI © 2025 — Unified AI Intelligence Hub •{' '}
          <span className="text-gray-600">8 Modeli AI • Synteza Synergiczna • Jedno Pole Tekstowe</span>
        </p>
      </footer>

      <style>{`
        @keyframes scanLine {
          0% { transform: translateX(-100%); }
          100% { transform: translateX(400%); }
        }

        .custom-scrollbar::-webkit-scrollbar {
          width: 4px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
          background: transparent;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
          background: rgba(139, 92, 246, 0.3);
          border-radius: 2px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
          background: rgba(139, 92, 246, 0.5);
        }
      `}</style>
    </div>
  );
}
