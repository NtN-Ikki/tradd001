# tradd001
<!DOCTYPE html>
<html lang="pt-BR">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Microservices Dashboard com IA</title>
    <script src="https://cdn.tailwindcss.com?plugins=typography"></script>
    <style>
      ::-webkit-scrollbar { height: 8px; width: 8px; }
      ::-webkit-scrollbar-thumb { background: rgb(156 163 175); border-radius: 4px; }
      .dark ::-webkit-scrollbar-thumb { background: rgb(55 65 81); }
      @keyframes fadeIn { from {opacity:0; transform: translateY(4px);} to {opacity:1; transform: translateY(0);} }
      @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.5; } }
      .ai-processing { animation: pulse 2s infinite; }
    </style>
  </head>
  <body class="bg-gray-50 dark:bg-gray-900 text-gray-800 dark:text-gray-100 transition-colors">
    <div id="root"></div>
  <script crossorigin src="https://cdn.jsdelivr.net/npm/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://cdn.jsdelivr.net/npm/react-dom@18/umd/react-dom.production.min.js"></script>
    <script crossorigin src="https://cdn.jsdelivr.net/npm/lucide-react@latest/dist/umd/lucide-react.production.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@babel/standalone/babel.min.js"></script>
    <script type="text/babel">
      const { 
        Globe, FileText, CheckCircle, Database, Zap, Link, 
        ArrowRight, Settings, Monitor, Search: SearchIcon, X: XIcon, Info,
        ExternalLink, Activity, Cpu, HardDrive, Wifi, Send, Copy, 
        RefreshCw, Languages, Edit3, Sparkles, AlertCircle, CheckCircle2
      } = lucideReact;
      /* -----------------------------
         Theme Context
      ------------------------------*/
      const ThemeContext = React.createContext();
      const ThemeProvider = ({ children }) => {
        const [darkMode, setDarkMode] = React.useState(false);
        React.useEffect(() => {
          document.documentElement.classList.toggle('dark', darkMode);
        }, [darkMode]);
        const toggle = React.useCallback(() => setDarkMode((m) => !m), []);
        const value = React.useMemo(() => ({ darkMode, toggle }), [darkMode, toggle]);
        return React.createElement(ThemeContext.Provider, { value }, children);
      };
      /* -----------------------------
         AI Service Configuration
      ------------------------------*/
      const AI_PROVIDERS = {
        gemini: {
          name: 'Google Gemini',
          endpoint: 'https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent',
          free: true,
          description: 'Até 1M tokens/minuto grátis'
        },
        groq: {
          name: 'Groq',
          endpoint: 'https://api.groq.com/openai/v1/chat/completions',
          free: true,
          description: 'Ultra-rápido, até 6k tokens/minuto'
        },
        huggingface: {
          name: 'Hugging Face',
          endpoint: 'https://api-inference.huggingface.co/models/',
          free: true,
          description: 'Modelos open-source gratuitos'
        }
      };
      /* -----------------------------
         AI Integration Component
      ------------------------------*/
      const AIProcessor = ({ onClose }) => {
        const [text, setText] = React.useState('');
        const [result, setResult] = React.useState('');
        const [loading, setLoading] = React.useState(false);
        const [provider, setProvider] = React.useState('gemini');
        const [task, setTask] = React.useState('translate');
        const [apiKey, setApiKey] = React.useState('');
        const [error, setError] = React.useState('');
        const processText = async () => {
          if (!text.trim()) return;
          if (!apiKey && provider !== 'huggingface') {
            setError('API Key é necessária para este provedor');
            return;
          }
          setLoading(true);
          setError('');
          try {
            let prompt = '';
            if (task === 'translate') {
              prompt = `Traduza o seguinte texto do português para o inglês (ou do inglês para o português, conforme apropriado). Retorne apenas a tradução:\n\n${text}`;
            } else {
              prompt = `Revise e melhore o seguinte texto, corrigindo gramática, fluência e clareza. Mantenha o tom original:\n\n${text}`;
            }
            let response;
            if (provider === 'gemini') {
              response = await fetch(`${AI_PROVIDERS.gemini.endpoint}?key=${apiKey}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                  contents: [{ parts: [{ text: prompt }] }],
                  generationConfig: { temperature: 0.3, maxOutputTokens: 1000 }
                })
              });
              const data = await response.json();
              setResult(data.candidates?.[0]?.content?.parts?.[0]?.text || 'Erro na resposta');        
            } else if (provider === 'groq') {
              response = await fetch(AI_PROVIDERS.groq.endpoint, {
                method: 'POST',
                headers: {
                  'Authorization': `Bearer ${apiKey}`,
                  'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                  model: 'llama3-8b-8192',
                  messages: [{ role: 'user', content: prompt }],
                  temperature: 0.3,
                  max_tokens: 1000
                })
              });
              const data = await response.json();
              setResult(data.choices?.[0]?.message?.content || 'Erro na resposta');            
            } else {
              // Hugging Face - usando modelo gratuito
              const model = task === 'translate' ? 'Helsinki-NLP/opus-mt-pt-en' : 'microsoft/DialoGPT-medium';
              response = await fetch(`${AI_PROVIDERS.huggingface.endpoint}${model}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ inputs: text })
              });
              const data = await response.json();
              setResult(data[0]?.translation_text || data[0]?.generated_text || 'Processamento concluído');
            }    
          } catch (err) {
            setError(`Erro: ${err.message}`);
            // Simulação para demonstração quando API falha
            setTimeout(() => {
              if (task === 'translate') {
                setResult(`[DEMO] Tradução simulada de: "${text}"`);
              } else {
                setResult(`[DEMO] Revisão simulada: "${text}" → Texto corrigido e melhorado.`);
              }
            }, 1500);
          }
          setLoading(false);
        };
        const copyResult = () => {
          navigator.clipboard.writeText(result);
        };
        return (
          <div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
            <div className="bg-white dark:bg-gray-800 rounded-xl p-6 w-full max-w-4xl max-h-[90vh] overflow-y-auto">
              <div className="flex items-center justify-between mb-6">
                <div className="flex items-center gap-2">
                  <Sparkles className="w-6 h-6 text-purple-500" />
                  <h2 className="text-xl font-bold">Processamento com IA</h2>
                </div>
                <button onClick={onClose} className="p-2 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-lg">
                  <XIcon className="w-5 h-5" />
                </button>
              </div>
              {/* Configuração */}
              <div className="grid md:grid-cols-3 gap-4 mb-6">
                <div>
                  <label className="block text-sm font-medium mb-2">Provedor de IA</label>
                  <select 
                    value={provider} 
                    onChange={(e) => setProvider(e.target.value)}
                    className="w-full p-2 border rounded-lg dark:bg-gray-700 dark:border-gray-600"
                  >
                    {Object.entries(AI_PROVIDERS).map(([key, p]) => (
                      <option key={key} value={key}>{p.name}</option>
                    ))}
                  </select>
                  <p className="text-xs text-gray-500 mt-1">{AI_PROVIDERS[provider].description}</p>
                </div>
                <div>
                  <label className="block text-sm font-medium mb-2">Tarefa</label>
                  <select 
                    value={task} 
                    onChange={(e) => setTask(e.target.value)}
                    className="w-full p-2 border rounded-lg dark:bg-gray-700 dark:border-gray-600"
                  >
                    <option value="translate">Tradução</option>
                    <option value="revision">Revisão</option>
                  </select>
                </div>
                <div>
                  <label className="block text-sm font-medium mb-2">
                    API Key
                    {provider === 'huggingface' && <span className="text-green-500 text-xs ml-1">(Opcional)</span>}
                  </label>
                  <input
                    type="password"
                    value={apiKey}
                    onChange={(e) => setApiKey(e.target.value)}
                    placeholder={provider === 'huggingface' ? 'Opcional para HF' : 'Cole sua API key aqui'}
                    className="w-full p-2 border rounded-lg dark:bg-gray-700 dark:border-gray-600"
                  />
                </div>
              </div>
              {/* Interface de Processamento */}
              <div className="grid md:grid-cols-2 gap-6">
                <div>
                  <label className="block text-sm font-medium mb-2 flex items-center gap-2">
                    {task === 'translate' ? <Languages className="w-4 h-4" /> : <Edit3 className="w-4 h-4" />}
                    Texto Original
                  </label>
                  <textarea
                    value={text}
                    onChange={(e) => setText(e.target.value)}
                    placeholder={task === 'translate' ? 'Cole o texto para traduzir...' : 'Cole o texto para revisar...'}
                    className="w-full h-40 p-3 border rounded-lg resize-none dark:bg-gray-700 dark:border-gray-600"
                  />
                  <div className="flex justify-between items-center mt-2">
                    <span className="text-xs text-gray-500">{text.length} caracteres</span>
                    <button
                      onClick={processText}
                      disabled={loading || !text.trim()}
                      className="flex items-center gap-2 px-4 py-2 bg-purple-500 text-white rounded-lg hover:bg-purple-600 disabled:opacity-50 disabled:cursor-not-allowed"
                    >
                      {loading ? (
                        <>
                          <RefreshCw className="w-4 h-4 animate-spin" />
                          Processando...
                        </>
                      ) : (
                        <>
                          <Send className="w-4 h-4" />
                          {task === 'translate' ? 'Traduzir' : 'Revisar'}
                        </>
                      )}
                    </button>
                  </div>
                </div>
                <div>
                  <label className="block text-sm font-medium mb-2 flex items-center gap-2">
                    <CheckCircle2 className="w-4 h-4" />
                    Resultado
                  </label>
                  <div className="relative">
                    <textarea
                      value={result}
                      readOnly
                      placeholder="O resultado aparecerá aqui..."
                      className={`w-full h-40 p-3 border rounded-lg resize-none bg-gray-50 dark:bg-gray-900 dark:border-gray-600 ${loading ? 'ai-processing' : ''}`}
                    />
                    {result && (
                      <button
                        onClick={copyResult}
                        className="absolute top-2 right-2 p-1 hover:bg-gray-200 dark:hover:bg-gray-700 rounded"
                        title="Copiar resultado"
                      >
                        <Copy className="w-4 h-4" />
                      </button>
                    )}
                  </div>
                  <div className="flex justify-between items-center mt-2">
                    <span className="text-xs text-gray-500">{result.length} caracteres</span>
                    {result && (
                      <span className="text-xs text-green-600 dark:text-green-400">✓ Processado com sucesso</span>
                    )}
                  </div>
                </div>
              </div>
              {/* Mensagens de Estado */}
              {error && (
                <div className="mt-4 p-3 bg-red-50 dark:bg-red-900/20 border border-red-200 dark:border-red-800 rounded-lg flex items-center gap-2">
                  <AlertCircle className="w-4 h-4 text-red-500" />
                  <span className="text-red-700 dark:text-red-300">{error}</span>
                </div>
              )}
              {/* Guia de Setup */}
              <div className="mt-6 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-lg">
                <h3 className="font-medium mb-2 flex items-center gap-2">
                  <Info className="w-4 h-4" />
                  Como obter API Keys gratuitas:
                </h3>
                <div className="text-sm space-y-1">
                  <div><strong>Google Gemini:</strong> <a href="https://aistudio.google.com/app/apikey" target="_blank" className="text-blue-600 hover:underline">AI Studio</a> - 1M tokens/minuto grátis</div>
                  <div><strong>Groq:</strong> <a href="https://console.groq.com/keys" target="_blank" className="text-blue-600 hover:underline">Groq Console</a> - 6k tokens/minuto grátis</div>
                  <div><strong>Hugging Face:</strong> <a href="https://huggingface.co/settings/tokens" target="_blank" className="text-blue-600 hover:underline">HF Tokens</a> - Grátis (opcional)</div>
                </div>
              </div>
            </div>
          </div>
        );
      };
      /* -----------------------------
         Service Card (Updated)
      ------------------------------*/
      const ServiceCard = React.memo(({ service, selected, onSelect, onAIProcess }) => {
        const [localStatus, setLocalStatus] = React.useState('online');
        React.useEffect(() => {
          const statuses = ['online','online','online','degraded','online'];
          const id = setInterval(() => setLocalStatus(statuses[Math.floor(Math.random()*statuses.length)]), 5000);
          return () => clearInterval(id);
        }, []);
        const handleKeyDown = React.useCallback((e) => {
          if (e.key === 'Enter' || e.key === ' ') {
            e.preventDefault();
            onSelect(service.id);
          }
        }, [service.id, onSelect]);
        const icons = { Globe, FileText, CheckCircle, Database, Zap, Link, Monitor, Activity, Cpu, HardDrive, Wifi };
        const Icon = icons[service.icon];
        const canUseAI = service.id === 'translation' || service.id === 'revision';
        return (
          <div
            role="button"
            tabIndex={0}
            className={`border rounded-xl p-4 transition hover:shadow-md cursor-pointer ${selected ? 'bg-indigo-50 dark:bg-indigo-900/30' : 'bg-white dark:bg-neutral-800'}`}
            onClick={() => onSelect(service.id)}
            onKeyDown={handleKeyDown}
          >
            <div className="flex items-start gap-3">
              <div className={`p-2 rounded-md ${service.color}`}>
                {Icon && <Icon className="w-6 h-6 text-white" />}
              </div>
              <div className="flex-1">
                <h3 className="font-semibold">{service.name}</h3>
                <p className="text-sm text-gray-600 dark:text-gray-300">{service.description}</p>
                <div className="flex items-center justify-between mt-2">
                  <div className="flex items-center space-x-2">
                    <div className={`w-2 h-2 rounded-full ${localStatus === 'online' ? 'bg-green-500 animate-pulse' : localStatus === 'degraded' ? 'bg-yellow-500 animate-pulse' : 'bg-red-500'}`}></div>
                    <span className="text-xs text-gray-600 dark:text-gray-300">
                      {localStatus === 'online' ? 'Online' : localStatus === 'degraded' ? 'Degradado' : 'Offline'}
                    </span>
                  </div>
                  {canUseAI && (
                    <button
                      onClick={(e) => {
                        e.stopPropagation();
                        onAIProcess();
                      }}
                      className="flex items-center gap-1 px-2 py-1 bg-purple-500 text-white text-xs rounded hover:bg-purple-600"
                    >
                      <Sparkles className="w-3 h-3" />
                      Testar IA
                    </button>
                  )}
                </div>
              </div>
            </div>
            {selected && (
              <div className="mt-4 space-y-4">
                <section>
                  <h4 className="font-medium mb-1">Responsabilidades</h4>
                  <ul>
                    {service.responsibilities.map((r,i) => (
                      {% raw %}
                       <li key={i} className="pl-4 relative text-sm mb-1 animate-[fadeIn_0.3s_both]" style={{'--delay': `${i*80}ms`}}>
                         {% endraw %}
                        <span className="absolute left-0 top-1.5 w-1 h-1 rounded-full bg-indigo-500"></span>
                        {r}
                      </li>
                    ))}
                  </ul>
                </section>
                <section>
                  <h4 className="font-medium mb-1">Tech Stack</h4>
                  <div className="flex flex-wrap gap-2">
                    {service.tech.split(',').map(t => t.trim()).map((t,i)=>(
                      <span key={i} className="px-2 py-0.5 bg-neutral-200 dark:bg-neutral-700 rounded text-xs">{t}</span>
                    ))}
                  </div>
                </section>
                {canUseAI && (
                  <section className="p-3 bg-gradient-to-r from-purple-50 to-pink-50 dark:from-purple-900/20 dark:to-pink-900/20 rounded-lg">
                    <h4 className="font-medium mb-1 flex items-center gap-2">
                      <Sparkles className="w-4 h-4 text-purple-500" />
                      IA Integrada
                    </h4>
                    <p className="text-sm text-gray-600 dark:text-gray-300 mb-2">
                      Este serviço agora suporta processamento com IA usando provedores gratuitos.
                    </p>
                    <button
                      onClick={(e) => {
                        e.stopPropagation();
                        onAIProcess();
                      }}
                      className="w-full flex items-center justify-center gap-2 px-3 py-2 bg-gradient-to-r from-purple-500 to-pink-500 text-white rounded hover:from-purple-600 hover:to-pink-600"
                    >
                      <Sparkles className="w-4 h-4" />
                      Abrir Interface de IA
                    </button>
                  </section>
                )}
              </div>
            )}
          </div>
        );
      });
      /* -----------------------------
         Main Component
      ------------------------------*/
      const MicroservicesArchitecture = () => {
        const [selectedService, setSelectedService] = React.useState(null);
        const [search, setSearch] = React.useState('');
        const [showAI, setShowAI] = React.useState(false);
        const { darkMode, toggle } = React.useContext(ThemeContext);
        const services = React.useMemo(() => [
          {
            id: 'api-gateway',
            name: 'API Gateway',
            icon: 'Monitor',
            color: 'bg-blue-500',
            description: 'Ponto de entrada único, autenticação, rate limiting',
            responsibilities: [
              'Roteamento de requisições',
              'Autenticação JWT',
              'Rate limiting',
              'Logging e monitoramento',
              'Circuit breaker',
              'Load balancing'
            ],
            tech: 'Kong, Nginx, AWS API Gateway, Envoy'
          },
          {
            id: 'web-scraper',
            name: 'Web Scraper',
            icon: 'Link',
            color: 'bg-green-500',
            description: 'Extrai conteúdo de websites',
            responsibilities: [
              'Validação de URLs',
              'Extração de texto limpo',
              'Detecção de idioma',
              'Cache de conteúdo',
              'Anti‑bot detection',
              'Parallel processing'
            ],
            tech: 'Puppeteer, Cheerio, Playwright'
          },
          {
            id: 'translation',
            name: 'Translation + IA',
            icon: 'Globe',
            color: 'bg-purple-500',
            description: 'Tradução com IA - Gemini, Groq, HuggingFace',
            responsibilities: [
              'Tradução EN‑US ↔ PT‑BR com IA',
              'Detecção automática de idioma',
              'Múltiplos provedores de IA',
              'Cache de traduções',
              'Processamento em lote',
              'Comparação de qualidade'
            ],
            tech: 'Google Gemini API, Groq API, HuggingFace, OpenAI (opcional)'
          },
          {
            id: 'revision',
            name: 'Text Revision + IA',
            icon: 'CheckCircle',
            color: 'bg-orange-500',
            description: 'Revisão inteligente com IA gratuita',
            responsibilities: [
              'Correção gramatical com IA',
              'Melhoria de fluência',
              'Análise de contexto',
              'Formatação inteligente',
              'Consistência de estilo',
              'Métricas de legibilidade'
            ],
            tech: 'Google Gemini, Groq LLaMA, HuggingFace Models, GPT (opcional)'
          },
          {
            id: 'orchestrator',
            name: 'Orchestrator',
            icon: 'Zap',
            color: 'bg-red-500',
            description: 'Coordena o fluxo de trabalho',
            responsibilities: [
              'Gerenciamento de workflow',
              'Controle de status',
              'Retry logic',
              'Notificações',
              'Job scheduling',
              'Error handling'
            ],
            tech: 'Node.js, Bull Queue, Redis, Temporal'
          },
          {
            id: 'database',
            name: 'Database',
            icon: 'Database',
            color: 'bg-gray-600',
            description: 'Persistência de dados',
            responsibilities: [
              'Histórico de traduções',
              'Cache de resultados de IA',
              'Métricas de uso',
              'Configurações de usuário',
              'Backup automation',
              'Data archiving'
            ],
            tech: 'PostgreSQL, Redis, MongoDB, InfluxDB'
          },
        ], []);
        const filtered = React.useMemo(() => {
          if (!search) return services;
          const q = search.toLowerCase();
          return services.filter(s => 
            s.name.toLowerCase().includes(q) ||
            s.description.toLowerCase().includes(q) ||
            s.tech.toLowerCase().includes(q)
          );
        }, [search, services]);
        const handleSelect = React.useCallback((id) => {
          setSelectedService(prev => prev === id ? null : id);
        }, []);
        return (
          <main className="max-w-5xl mx-auto p-6 space-y-6">
            {/* Header */}
            <header className="flex items-center justify-between">
              <div>
                <h1 className="text-2xl font-bold flex items-center gap-2">
                  Microsserviços + IA
                  <Sparkles className="w-6 h-6 text-purple-500" />
                </h1>
                <p className="text-sm text-gray-600 dark:text-gray-400">
                  Tradução e Revisão com IA Gratuita Integrada
                </p>
              </div>
              <div className="flex items-center gap-2">
                <button 
                  onClick={() => setShowAI(true)}
                  className="flex items-center gap-2 px-3 py-2 bg-gradient-to-r from-purple-500 to-pink-500 text-white rounded-lg hover:from-purple-600 hover:to-pink-600"
                >
                  <Sparkles className="w-4 h-4" />
                  Testar IA
                </button>
                <button 
                  onClick={toggle}
                  title={darkMode ? 'Modo Claro' : 'Modo Escuro'}
                  className="text-lg p-2 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg"
                >
                  {darkMode ? '☀️' : '🌙'}
                </button>
              </div>
            </header>
            {/* Search */}
            <section>
              <div className="relative max-w-md">
                <SearchIcon className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-400" />
                <input
                  type="text"
                  placeholder="Buscar microsserviços..."
                  value={search}
                  onChange={(e)=>setSearch(e.target.value)}
                  className="w-full pl-10 pr-10 py-2 bg-white dark:bg-neutral-800 border border-gray-300 dark:border-neutral-700 rounded-md focus:outline-none"
                />
                {search && (
                  <button
                    onClick={()=>setSearch('')}
                    className="absolute right-3 top-1/2 -translate-y-1/2"
                  >
                    <XIcon className="w-4 h-4 text-gray-400" />
                  </button>
                )}
              </div>
            </section>
            {/* Grid */}
            <section className="grid sm:grid-cols-2 lg:grid-cols-3 gap-4">
              {filtered.map(s => (
                <ServiceCard
                  key={s.id}
                  service={s}
                  selected={selectedService===s.id}
                  onSelect={handleSelect}
                  onAIProcess={() => setShowAI(true)}
                />
              ))}
            </section>
            {/* AI Modal */}
            {showAI && <AIProcessor onClose={() => setShowAI(false)} />}
            {/* Footer */}
            <footer className="text-center text-xs text-gray-500 dark:text-gray-400 pt-8">
              <div className="flex items-center justify-center gap-2 mb-2">
                <Sparkles className="w-4 h-4" />
                Powered by Free AI APIs - Gemini, Groq, HuggingFace
              </div>
              Deploy gratuito no GitHub Pages, Netlify ou Vercel 🚀
            </footer>
          </main>
        );
      };
      /* -----------------------------
         Render
      ------------------------------*/
      const root = ReactDOM.createRoot(document.getElementById('root'));
      root.render(
        <ThemeProvider>
          <MicroservicesArchitecture />
        </ThemeProvider>
      );
    </script>
  </body>
</html>
