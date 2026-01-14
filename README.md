<html lang="vi" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>VPN IPSEC Demo</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&amp;family=Space+Grotesk:wght@400;500;600;700&amp;display=swap" rel="stylesheet">
  <style>
    body {
      box-sizing: border-box;
    }
    
    * {
      font-family: 'Space Grotesk', sans-serif;
    }
    
    .mono {
      font-family: 'JetBrains Mono', monospace;
    }
    
    @keyframes pulse-glow {
      0%, 100% { box-shadow: 0 0 20px rgba(34, 197, 94, 0.3); }
      50% { box-shadow: 0 0 40px rgba(34, 197, 94, 0.6); }
    }
    
    @keyframes data-flow {
      0% { transform: translateX(-100%); opacity: 0; }
      50% { opacity: 1; }
      100% { transform: translateX(100%); opacity: 0; }
    }
    
    @keyframes typing {
      from { width: 0; }
      to { width: 100%; }
    }
    
    @keyframes blink {
      0%, 50% { border-color: #22c55e; }
      51%, 100% { border-color: transparent; }
    }
    
    .connected {
      animation: pulse-glow 2s ease-in-out infinite;
    }
    
    .data-packet {
      animation: data-flow 2s linear infinite;
    }
    
    .terminal-text {
      overflow: hidden;
      white-space: nowrap;
      border-right: 2px solid #22c55e;
      animation: typing 2s steps(40) forwards, blink 0.8s step-end infinite;
    }
    
    .gradient-bg {
      background: linear-gradient(135deg, #0f172a 0%, #1e293b 50%, #0f172a 100%);
    }
    
    .glass-card {
      background: rgba(30, 41, 59, 0.8);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(71, 85, 105, 0.5);
    }
    
    .tunnel-line {
      background: linear-gradient(90deg, transparent, #22c55e, transparent);
      height: 2px;
    }
    
    @keyframes tunnel-flow {
      0% { background-position: -200% 0; }
      100% { background-position: 200% 0; }
    }
    
    .tunnel-active {
      background: linear-gradient(90deg, transparent 0%, #22c55e 50%, transparent 100%);
      background-size: 200% 100%;
      animation: tunnel-flow 1.5s linear infinite;
    }
    
    .phase-indicator {
      transition: all 0.5s ease;
    }
    
    .phase-complete {
      background: #22c55e;
      box-shadow: 0 0 10px rgba(34, 197, 94, 0.5);
    }
    
    .phase-active {
      background: #eab308;
      box-shadow: 0 0 15px rgba(234, 179, 8, 0.5);
      animation: pulse 1s ease-in-out infinite;
    }
    
    @keyframes pulse {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.1); }
    }
    
    .log-entry {
      animation: fadeIn 0.3s ease-out;
    }
    
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(-10px); }
      to { opacity: 1; transform: translateY(0); }
    }
    
    .animate-fadeIn {
      animation: fadeIn 0.3s ease-out;
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
 </head>
 <body class="h-full gradient-bg text-white overflow-auto">
  <div class="min-h-full p-4 md:p-6"><!-- Header -->
   <div class="text-center mb-6">
    <h1 id="main-title" class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-green-400 to-emerald-500 bg-clip-text text-transparent mb-2">🔐 VPN IPSEC Authentication Demo</h1>
    <p id="subtitle" class="text-slate-400 text-sm md:text-base">Mô phỏng quy trình thiết lập kết nối VPN với xác thực IPSEC</p>
   </div><!-- Decrypt Modal -->
   <div id="decrypt-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center z-50 hidden">
    <div class="glass-card rounded-2xl p-6 w-full max-w-lg mx-4">
     <div class="flex items-center justify-between mb-4">
      <h2 class="text-xl font-bold text-amber-400 flex items-center gap-2">
       <svg class="w-6 h-6" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 11V7a4 4 0 118 0m-4 8v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2z" />
       </svg> Giải mã tin nhắn</h2><button onclick="closeDecryptModal()" class="text-slate-400 hover:text-white">
       <svg class="w-6 h-6" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
       </svg></button>
     </div>
     <div class="bg-slate-900 rounded-xl p-4 mb-4">
      <div class="text-xs text-slate-400 mb-2">
       Tin nhắn đã mã hóa (Ciphertext)
      </div>
      <div id="decrypt-ciphertext" class="mono text-xs text-purple-400 break-all max-h-32 overflow-y-auto"></div>
     </div>
     <form id="decrypt-form" onsubmit="attemptDecryption(event)" class="space-y-4">
      <div class="text-sm text-slate-300 mb-3">
       <div class="flex items-center gap-2 mb-2">
        <svg class="w-4 h-4 text-amber-400" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
        </svg><span class="font-semibold text-amber-400">Chọn thuật toán giải mã</span>
       </div>
       <p class="text-xs text-slate-400">Bạn c�����n chọn đúng các tham số mã hóa mà người gửi đã sử dụng</p>
      </div>
      <div><label class="text-slate-400 text-xs flex items-center gap-1.5 mb-2">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
        </svg> Encryption Algorithm </label> <select id="decrypt-encryption" class="w-full bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-amber-500"> <option value="AES-256-CBC">AES-256-CBC</option> <option value="AES-128-CBC">AES-128-CBC</option> <option value="3DES">3DES</option> </select>
      </div>
      <div><label class="text-slate-400 text-xs flex items-center gap-1.5 mb-2">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zm0 0h12a2 2 0 002-2v-4a2 2 0 00-2-2h-2.343M11 7.343l1.657-1.657a2 2 0 012.828 0l2.829 2.829a2 2 0 010 2.828l-8.486 8.485M7 17h.01" />
        </svg> Hash Algorithm </label> <select id="decrypt-hash" class="w-full bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-amber-500"> <option value="sha256">SHA-256</option> <option value="sha1">SHA-1</option> <option value="md5">MD5</option> </select>
      </div>
      <div><label class="text-slate-400 text-xs flex items-center gap-1.5 mb-2">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z" />
        </svg> DH Group </label> <select id="decrypt-dhgroup" class="w-full bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-amber-500"> <option value="14">Group 14 (2048-bit)</option> <option value="5">Group 5 (1536-bit)</option> <option value="2">Group 2 (1024-bit)</option> </select>
      </div>
      <div><label class="text-slate-400 text-xs flex items-center gap-1.5 mb-2">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
        </svg> Authentication </label> <select id="decrypt-auth" class="w-full bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-amber-500"> <option value="psk">Pre-Shared Key</option> <option value="cert">X.509 Certificate</option> </select>
      </div>
      <div id="decrypt-error" class="hidden bg-red-500/10 border border-red-500/30 rounded-lg px-3 py-2 text-sm text-red-400 flex items-center gap-2">
       <svg class="w-4 h-4" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
       </svg><span>Sai tham số giải mã! Vui lòng thử lại.</span>
      </div>
      <div id="decrypt-success" class="hidden bg-green-500/10 border border-green-500/30 rounded-lg p-4">
       <div class="text-xs text-slate-400 mb-2 flex items-center gap-1.5">
        <svg class="w-4 h-4 text-green-400" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg> Tin nhắn gốc (Plaintext)
       </div>
       <div id="decrypt-plaintext" class="text-sm text-green-400 font-medium"></div>
      </div>
      <div class="flex gap-3"><button type="submit" class="flex-1 bg-gradient-to-r from-amber-500 to-orange-600 hover:from-amber-600 hover:to-orange-700 text-white font-semibold py-3 px-4 rounded-xl transition-all duration-300 flex items-center justify-center gap-2">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 11V7a4 4 0 118 0m-4 8v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2z" />
        </svg> Giải mã </button> <button type="button" onclick="closeDecryptModal()" class="bg-slate-700 hover:bg-slate-600 text-white font-semibold py-3 px-6 rounded-xl transition-all duration-300"> Đóng </button>
      </div>
     </form>
    </div>
   </div><!-- Login Modal -->
   <div id="login-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center z-50">
    <div class="glass-card rounded-2xl p-6 w-full max-w-md mx-4">
     <div class="text-center mb-6">
      <div class="text-4xl mb-3">
       🔐
      </div>
      <h2 class="text-2xl font-bold text-green-400 mb-2">VPN IPSEC Login</h2>
      <p class="text-slate-400 text-sm">Đăng nhập để tiếp tục</p>
     </div>
     <form id="login-form" onsubmit="handleLogin(event)" class="space-y-4">
      <div><label for="login-username" class="block text-sm text-slate-300 mb-2">Tên đăng nhập</label> <input type="text" id="login-username" required placeholder="Nhập tên đăng nhập" class="w-full bg-slate-800 border border-slate-600 rounded-lg px-4 py-2.5 text-white focus:outline-none focus:border-green-500">
      </div>
      <div><label for="login-password" class="block text-sm text-slate-300 mb-2">Mật khẩu</label> <input type="password" id="login-password" required placeholder="Nhập mật khẩu" class="w-full bg-slate-800 border border-slate-600 rounded-lg px-4 py-2.5 text-white focus:outline-none focus:border-green-500">
      </div>
      <div id="login-error" class="hidden text-sm text-red-400 bg-red-500/10 border border-red-500/30 rounded-lg px-3 py-2">
       Tên đăng nhập hoặc mật khẩu không đúng!
      </div><button type="submit" class="w-full bg-gradient-to-r from-green-500 to-emerald-600 hover:from-green-600 hover:to-emerald-700 text-white font-semibold py-3 px-4 rounded-xl transition-all duration-300"> Đăng nhập </button>
     </form>
     <div class="mt-6 pt-6 border-t border-slate-700">
      <div class="text-xs text-slate-400 space-y-1">
       <div>
        <strong>Remote Client:</strong> user / user123
       </div>
       <div>
        <strong>VPN Gateway:</strong> VPN / VPN123
       </div>
      </div>
     </div>
    </div>
   </div><!-- Main Demo Area -->
   <div id="main-content" class="max-w-6xl mx-auto hidden"><!-- User Info Bar -->
    <div class="glass-card rounded-xl p-3 mb-4 flex items-center justify-between">
     <div class="flex items-center gap-3">
      <div id="user-avatar" class="text-2xl"></div>
      <div>
       <div class="text-sm font-semibold text-white" id="logged-user-name"></div>
       <div class="text-xs text-slate-400" id="logged-user-role"></div>
      </div>
     </div><button onclick="logout()" class="text-xs text-red-400 hover:text-red-300 transition-colors"> Đăng xuất </button>
    </div><!-- Network Topology -->
    <div class="glass-card rounded-2xl p-4 md:p-6 mb-6">
     <h2 class="text-lg font-semibold text-green-400 mb-4 flex items-center gap-2">
      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 3v2m6-2v2M9 19v2m6-2v2M5 9H3m2 6H3m18-6h-2m2 6h-2M7 19h10a2 2 0 002-2V7a2 2 0 00-2-2H7a2 2 0 00-2 2v10a2 2 0 002 2zM9 9h6v6H9V9z" />
      </svg> Network Topology</h2>
     <div class="flex flex-col md:flex-row items-center justify-between gap-4"><!-- Client Side -->
      <div id="client-node" class="glass-card rounded-xl p-4 w-full md:w-48 text-center transition-all duration-500">
       <div class="text-4xl mb-2">
        💻
       </div>
       <div class="font-semibold text-white">
        Remote Client
       </div>
       <div class="text-xs text-slate-400 mono mt-1">
        192.168.1.100
       </div>
       <div id="client-status" class="text-xs mt-2 px-2 py-1 rounded-full bg-slate-700 text-slate-300">
        Disconnected
       </div>
      </div><!-- VPN Tunnel -->
      <div class="flex-1 w-full md:w-auto px-4 py-6 relative">
       <div class="text-center text-xs text-slate-500 mb-2">
        IPSEC Tunnel
       </div>
       <div id="tunnel-line" class="h-1 rounded-full bg-slate-700 relative overflow-hidden">
        <div id="tunnel-flow" class="absolute inset-0 opacity-0 tunnel-active"></div>
       </div>
       <div id="tunnel-packets" class="flex justify-center gap-1 mt-2 h-6 overflow-hidden"><!-- Data packets will appear here -->
       </div>
       <div class="flex justify-between text-xs text-slate-500 mt-2"><span>�� ESP Encrypted</span> <span id="tunnel-cipher-display">AES-256</span>
       </div>
      </div><!-- Server Side -->
      <div id="server-node" class="glass-card rounded-xl p-4 w-full md:w-48 text-center transition-all duration-500">
       <div class="text-4xl mb-2">
        🖥️
       </div>
       <div class="font-semibold text-white">
        VPN Gateway
       </div>
       <div class="text-xs text-slate-400 mono mt-1">
        10.0.0.1
       </div>
       <div id="server-status" class="text-xs mt-2 px-2 py-1 rounded-full bg-slate-700 text-slate-300">
        Listening
       </div>
      </div>
     </div>
    </div><!-- IPSEC Phases -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6"><!-- Phase 1: IKE -->
     <div class="glass-card rounded-2xl p-4">
      <div class="flex items-center gap-3 mb-4">
       <div id="phase1-indicator" class="w-4 h-4 rounded-full bg-slate-600 phase-indicator"></div>
       <h3 class="font-semibold text-amber-400">Phase 1: IKE (Internet Key Exchange)</h3>
      </div>
      <div class="space-y-2 text-sm">
       <div id="ike-step1" class="flex items-center gap-2 text-slate-500"><span class="w-5 h-5 rounded-full bg-slate-700 flex items-center justify-center text-xs">1</span> <span>SA Proposal Exchange</span>
       </div>
       <div id="ike-step2" class="flex items-center gap-2 text-slate-500"><span class="w-5 h-5 rounded-full bg-slate-700 flex items-center justify-center text-xs">2</span> <span>Diffie-Hellman Key Exchange</span>
       </div>
       <div id="ike-step3" class="flex items-center gap-2 text-slate-500"><span class="w-5 h-5 rounded-full bg-slate-700 flex items-center justify-center text-xs">3</span> <span>Authentication (PSK/Certificate)</span>
       </div>
       <div id="ike-step4" class="flex items-center gap-2 text-slate-500"><span class="w-5 h-5 rounded-full bg-slate-700 flex items-center justify-center text-xs">4</span> <span>IKE SA Established</span>
       </div>
      </div>
     </div><!-- Phase 2: IPSEC -->
     <div class="glass-card rounded-2xl p-4">
      <div class="flex items-center gap-3 mb-4">
       <div id="phase2-indicator" class="w-4 h-4 rounded-full bg-slate-600 phase-indicator"></div>
       <h3 class="font-semibold text-emerald-400">Phase 2: IPSEC SA Negotiation</h3>
      </div>
      <div class="space-y-2 text-sm">
       <div id="ipsec-step1" class="flex items-center gap-2 text-slate-500"><span class="w-5 h-5 rounded-full bg-slate-700 flex items-center justify-center text-xs">1</span> <span>Quick Mode Initiation</span>
       </div>
       <div id="ipsec-step2" class="flex items-center gap-2 text-slate-500"><span class="w-5 h-5 rounded-full bg-slate-700 flex items-center justify-center text-xs">2</span> <span>IPSEC SA Parameters</span>
       </div>
       <div id="ipsec-step3" class="flex items-center gap-2 text-slate-500"><span class="w-5 h-5 rounded-full bg-slate-700 flex items-center justify-center text-xs">3</span> <span>Session Keys Generated</span>
       </div>
       <div id="ipsec-step4" class="flex items-center gap-2 text-slate-500"><span class="w-5 h-5 rounded-full bg-slate-700 flex items-center justify-center text-xs">4</span> <span>Tunnel Ready</span>
       </div>
      </div>
     </div>
    </div><!-- Control Panel & Logs -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4"><!-- Configuration -->
     <div class="glass-card rounded-2xl p-4">
      <h3 class="font-semibold text-blue-400 mb-4 flex items-center gap-2">
       <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" /> <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
       </svg> IPSEC Configuration</h3>
      <div class="space-y-3 text-sm">
       <div><label class="text-slate-400 text-xs">Encryption Algorithm</label> <select id="encryption" class="w-full mt-1 bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-green-500"> <option value="AES-256-CBC">AES-256-CBC</option> <option value="AES-128-CBC">AES-128-CBC</option> <option value="3DES">3DES</option> </select>
       </div>
       <div><label class="text-slate-400 text-xs">Hash Algorithm</label> <select id="hash" class="w-full mt-1 bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-green-500"> <option value="sha256">SHA-256</option> <option value="sha1">SHA-1</option> <option value="md5">MD5</option> </select>
       </div>
       <div><label class="text-slate-400 text-xs">DH Group</label> <select id="dhgroup" class="w-full mt-1 bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-green-500"> <option value="14">Group 14 (2048-bit)</option> <option value="5">Group 5 (1536-bit)</option> <option value="2">Group 2 (1024-bit)</option> </select>
       </div>
       <div><label class="text-slate-400 text-xs">Authentication</label> <select id="auth" class="w-full mt-1 bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-green-500"> <option value="psk">Pre-Shared Key</option> <option value="cert">X.509 Certificate</option> </select>
       </div>
      </div><button id="connect-btn" onclick="startVPNConnection()" class="w-full mt-4 bg-gradient-to-r from-green-500 to-emerald-600 hover:from-green-600 hover:to-emerald-700 text-white font-semibold py-3 px-4 rounded-xl transition-all duration-300 flex items-center justify-center gap-2">
       <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
       </svg><span>Connect VPN</span> </button> <button id="disconnect-btn" onclick="disconnectVPN()" class="w-full mt-2 bg-slate-700 hover:bg-slate-600 text-white font-semibold py-3 px-4 rounded-xl transition-all duration-300 hidden"> Disconnect </button>
     </div><!-- Connection Log -->
     <div class="glass-card rounded-2xl p-4 md:col-span-2">
      <h3 class="font-semibold text-purple-400 mb-4 flex items-center gap-2">
       <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
       </svg> Connection Log</h3>
      <div id="log-container" class="bg-slate-900 rounded-xl p-3 h-64 overflow-y-auto mono text-xs space-y-1">
       <div class="text-slate-500">
        [System] VPN client initialized. Ready to connect.
       </div>
      </div>
     </div>
    </div><!-- Packet Encryption Visualization -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-6"><!-- Encryption Demo -->
     <div class="glass-card rounded-2xl p-4">
      <h3 class="font-semibold text-cyan-400 mb-4 flex items-center gap-2">
       <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
       </svg> Packet Encryption Process</h3>
      <div class="space-y-3">
       <div class="bg-slate-800/50 rounded-xl p-3">
        <div class="text-xs text-slate-400 mb-2">
         Original Packet (Plaintext)
        </div>
        <div id="plaintext-packet" class="mono text-sm text-green-400 break-all">
         Waiting for message...
        </div>
       </div>
       <div class="flex justify-center">
        <div class="bg-amber-500/20 text-amber-400 px-3 py-1 rounded-full text-xs flex items-center gap-2">
         <svg class="w-4 h-4 animate-spin" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
         </svg><span>ESP Encryption</span>
        </div>
       </div>
       <div class="bg-slate-800/50 rounded-xl p-3">
        <div class="text-xs text-slate-400 mb-2">
         Encrypted Packet (Ciphertext)
        </div>
        <div id="encrypted-packet" class="mono text-xs text-purple-400 break-all">
         Waiting for message...
        </div>
       </div>
       <div class="grid grid-cols-3 gap-2 text-xs text-center mt-4">
        <div class="bg-slate-800/50 rounded-lg p-2">
         <div class="text-green-400 font-semibold" id="packet-count">
          0
         </div>
         <div class="text-slate-500">
          Packets
         </div>
        </div>
        <div class="bg-slate-800/50 rounded-lg p-2">
         <div class="text-blue-400 font-semibold" id="bytes-sent">
          0 B
         </div>
         <div class="text-slate-500">
          Sent
         </div>
        </div>
        <div class="bg-slate-800/50 rounded-lg p-2">
         <div class="text-amber-400 font-semibold mono text-xs" id="current-cipher">
          AES-256-CBC
         </div>
         <div class="text-slate-500">
          Cipher
         </div>
        </div>
       </div>
      </div>
     </div><!-- Messaging Interface -->
     <div class="glass-card rounded-2xl p-4">
      <h3 class="font-semibold text-emerald-400 mb-4 flex items-center gap-2">
       <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
       </svg> Secure Messaging (Ciphertext View)</h3>
      <div id="chat-messages" class="bg-slate-900 rounded-xl p-3 h-48 overflow-y-auto mb-3 space-y-2">
       <div class="text-center text-slate-500 text-xs">
        Kết nối VPN để gửi tin nhắn bảo mật
       </div>
      </div>
      <form id="message-form" onsubmit="sendMessage(event)" class="flex gap-2"><input type="text" id="message-input" placeholder="Nhập tin nhắn..." disabled class="flex-1 bg-slate-800 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-green-500 disabled:opacity-50 disabled:cursor-not-allowed"> <button type="submit" id="send-btn" disabled class="bg-green-500 hover:bg-green-600 disabled:bg-slate-700 disabled:cursor-not-allowed text-white px-4 py-2 rounded-lg transition-all duration-300 flex items-center gap-2">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" />
        </svg></button>
      </form>
      <div class="mt-3 text-xs text-slate-400 flex items-center gap-2">
       <svg class="w-4 h-4 text-green-400" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
       </svg><span>Hiển thị ciphertext - chỉ VPN gateway có thể giải mã</span>
      </div>
     </div>
    </div><!-- Decrypted Messages Archive (VPN Gateway Only) -->
    <div id="vpn-archive-section" class="glass-card rounded-2xl p-4 mt-6 hidden">
     <div class="flex items-center justify-between mb-4">
      <h3 class="font-semibold text-orange-400 flex items-center gap-2">
       <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 7a2 2 0 012 2m4 0a6 6 0 01-7.743 5.743L11 17H9v2H7v2H4a1 1 0 01-1-1v-2.586a1 1 0 01.293-.707l5.964-5.964A6 6 0 1121 9z" />
       </svg> VPN Gateway - Decrypted Messages Archive</h3>
      <div class="text-xs bg-orange-500/20 text-orange-400 px-3 py-1 rounded-full">
       🔓 Private View
      </div>
     </div>
     <div id="decrypted-archive" class="bg-slate-900 rounded-xl p-4 max-h-64 overflow-y-auto space-y-2 mb-4">
      <div class="text-center text-slate-500 text-xs">
       Không có tin nhắn đã giải mã
      </div>
     </div><!-- VPN Reply Form -->
     <div class="bg-slate-800/50 rounded-xl p-4 border border-orange-500/20">
      <h4 class="text-sm font-semibold text-orange-300 mb-3 flex items-center gap-2">
       <svg class="w-4 h-4" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 10h10a8 8 0 018 8v2M3 10l6 6m-6-6l6-6" />
       </svg> VPN Gateway Reply</h4>
      <form id="vpn-reply-form" onsubmit="sendVPNReply(event)" class="flex gap-2"><input type="text" id="vpn-reply-input" placeholder="Nhập câu trả lời tùy ��..." disabled class="flex-1 bg-slate-900 border border-slate-600 rounded-lg px-3 py-2 text-white text-sm focus:outline-none focus:border-orange-500 disabled:opacity-50 disabled:cursor-not-allowed"> <button type="submit" id="vpn-reply-btn" disabled class="bg-orange-500 hover:bg-orange-600 disabled:bg-slate-700 disabled:cursor-not-allowed text-white px-4 py-2 rounded-lg transition-all duration-300 flex items-center gap-2">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" />
        </svg></button>
      </form>
     </div>
     <div class="mt-3 text-xs text-slate-400 flex items-center gap-2">
      <svg class="w-4 h-4 text-orange-400" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
      </svg><span>Chỉ VPN Gateway có khóa riêng để giải mã và xem nội dung này</span>
     </div>
    </div><!-- Security Info -->
    <div class="glass-card rounded-2xl p-4 mt-6">
     <h3 class="font-semibold text-cyan-400 mb-4 flex items-center gap-2">
      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
      </svg> IPSEC Security Components</h3>
     <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
      <div class="bg-slate-800/50 rounded-xl p-4">
       <div class="flex items-center gap-3 mb-3">
        <div class="text-3xl">
         🔐
        </div>
        <div>
         <div class="text-sm font-semibold text-green-400">
          ESP (Encapsulating Security Payload)
         </div>
         <div class="text-xs text-slate-400">
          Protocol 50
         </div>
        </div>
       </div>
       <div class="text-xs text-slate-300 space-y-1">
        <div>
         ✓ Mã hóa dữ liệu (Encryption)
        </div>
        <div>
         ✓ Xác thực gói tin (Authentication)
        </div>
        <div>
         ✓ Bảo vệ chống replay attacks
        </div>
        <div class="text-slate-500 italic mt-2">
         Sử dụng: Bảo mật nội dung và toàn vẹn dữ liệu
        </div>
       </div>
      </div>
      <div class="bg-slate-800/50 rounded-xl p-4">
       <div class="flex items-center gap-3 mb-3">
        <div class="text-3xl">
         🔑
        </div>
        <div>
         <div class="text-sm font-semibold text-amber-400">
          IKE (Internet Key Exchange)
         </div>
         <div class="text-xs text-slate-400">
          UDP Port 500
         </div>
        </div>
       </div>
       <div class="text-xs text-slate-300 space-y-1">
        <div>
         ✓ Thiết lập SA (Security Association)
        </div>
        <div>
         ✓ Trao đổi khóa Diffie-Hellman
        </div>
        <div>
         ✓ Xác thực peer (PSK/Certificate)
        </div>
        <div class="text-slate-500 italic mt-2">
         Sử dụng: Thiết lập tunnel và quản lý khóa
        </div>
       </div>
      </div>
      <div class="bg-slate-800/50 rounded-xl p-4">
       <div class="flex items-center gap-3 mb-3">
        <div class="text-3xl">
         📋
        </div>
        <div>
         <div class="text-sm font-semibold text-blue-400">
          SA (Security Association)
         </div>
         <div class="text-xs text-slate-400">
          Unidirectional
         </div>
        </div>
       </div>
       <div class="text-xs text-slate-300 space-y-1">
        <div>
         ✓ Lưu trữ thông số bảo mật
        </div>
        <div>
         ✓ SPI (Security Parameter Index)
        </div>
        <div>
         ✓ Thuật toán mã hóa và hash
        </div>
        <div class="text-slate-500 italic mt-2">
         Sử dụng: Quản lý thông số kết nối IPSec
        </div>
       </div>
      </div>
      <div class="bg-slate-800/50 rounded-xl p-4">
       <div class="flex items-center gap-3 mb-3">
        <div class="text-3xl">
         ✅
        </div>
        <div>
         <div class="text-sm font-semibold text-purple-400">
          AH (Authentication Header)
         </div>
         <div class="text-xs text-slate-400">
          Protocol 51
         </div>
        </div>
       </div>
       <div class="text-xs text-slate-300 space-y-1">
        <div>
         ��� Xác thực nguồn dữ liệu
        </div>
        <div>
         ✓ Đảm bảo toàn vẹn (Integrity)
        </div>
        <div>
         ✓ Chống replay attacks
        </div>
        <div class="text-slate-500 italic mt-2">
         Sử dụng: Xác thực không mã hóa (ít dùng)
        </div>
       </div>
      </div>
     </div>
    </div>
   </div>
  </div>
  <script>
    const defaultConfig = {
      main_title: '🔐 VPN IPSEC Authentication Demo',
      subtitle: 'Mô phỏng quy trình thiết lập kết nối VPN với xác thực IPSEC',
      background_color: '#0f172a',
      surface_color: '#1e293b',
      text_color: '#e2e8f0',
      primary_color: '#22c55e',
      secondary_color: '#3b82f6',
      font_family: 'Space Grotesk'
    };

    let isConnected = false;
    let connectionInterval = null;
    let packetCount = 0;
    let totalBytes = 0;
    let currentUser = null;
    let currentEncryptionParams = {};
    let currentDecryptMsgId = null;
    
    // Message storage - persists across logins
    let messageHistory = [];
    let decryptedArchive = [];

    // Login handling
    function handleLogin(event) {
      event.preventDefault();
      
      const username = document.getElementById('login-username').value;
      const password = document.getElementById('login-password').value;
      const errorDiv = document.getElementById('login-error');
      
      // Check credentials
      if ((username === 'user' && password === 'user123') || (username === 'VPN' && password === 'VPN123')) {
        currentUser = {
          username: username,
          role: username === 'user' ? 'Remote Client' : 'VPN Gateway Administrator',
          isVPN: username === 'VPN'
        };
        
        // Hide login modal and show main content
        document.getElementById('login-modal').classList.add('hidden');
        document.getElementById('main-content').classList.remove('hidden');
        
        // Update user info
        document.getElementById('user-avatar').textContent = username === 'user' ? '💻' : '🖥️';
        document.getElementById('logged-user-name').textContent = username;
        document.getElementById('logged-user-role').textContent = currentUser.role;
        
        // Show VPN archive section if VPN user
        if (currentUser.isVPN) {
          document.getElementById('vpn-archive-section').classList.remove('hidden');
        }
        
        addLog(`[System] ${currentUser.role} logged in successfully`, 'success');
      } else {
        errorDiv.classList.remove('hidden');
      }
    }

    function logout() {
      // Disconnect VPN if connected
      if (isConnected) {
        disconnectVPN();
      }
      
      // Reset UI
      currentUser = null;
      document.getElementById('login-modal').classList.remove('hidden');
      document.getElementById('main-content').classList.add('hidden');
      document.getElementById('login-username').value = '';
      document.getElementById('login-password').value = '';
      document.getElementById('login-error').classList.add('hidden');
      document.getElementById('vpn-archive-section').classList.add('hidden');
    }

    function showDecryptModal(msgId, sender) {
      currentDecryptMsgId = msgId;
      
      // Find message in history
      const msg = messageHistory.find(m => m.ciphertext === document.querySelector(`[data-msg-id="${msgId}"]`)?.dataset.plaintext);
      
      if (!msg) {
        // Find from DOM
        const msgElement = document.querySelector(`[data-msg-id="${msgId}"]`);
        if (msgElement) {
          const ciphertextElement = msgElement.querySelector('.mono.break-all');
          if (ciphertextElement) {
            document.getElementById('decrypt-ciphertext').textContent = ciphertextElement.textContent;
          }
        }
      } else {
        document.getElementById('decrypt-ciphertext').textContent = msg.ciphertext;
      }
      
      // Reset form
      document.getElementById('decrypt-error').classList.add('hidden');
      document.getElementById('decrypt-success').classList.add('hidden');
      document.getElementById('decrypt-plaintext').textContent = '';
      
      // Show modal
      document.getElementById('decrypt-modal').classList.remove('hidden');
    }

    function closeDecryptModal() {
      document.getElementById('decrypt-modal').classList.add('hidden');
      currentDecryptMsgId = null;
    }

    function attemptDecryption(event) {
      event.preventDefault();
      
      const selectedEncryption = document.getElementById('decrypt-encryption').value;
      const selectedHash = document.getElementById('decrypt-hash').value;
      const selectedDH = document.getElementById('decrypt-dhgroup').value;
      const selectedAuth = document.getElementById('decrypt-auth').value;
      
      // Check if parameters match
      const isCorrect = 
        selectedEncryption === currentEncryptionParams.encryption &&
        selectedHash === currentEncryptionParams.hash &&
        selectedDH === currentEncryptionParams.dhgroup &&
        selectedAuth === currentEncryptionParams.auth;
      
      if (isCorrect) {
        // Find plaintext from message history
        const msgElement = document.querySelector(`[data-msg-id="${currentDecryptMsgId}"]`);
        const plaintext = msgElement?.dataset.plaintext || 'Tin nhắn đã được giải mã';
        
        document.getElementById('decrypt-plaintext').textContent = plaintext;
        document.getElementById('decrypt-success').classList.remove('hidden');
        document.getElementById('decrypt-error').classList.add('hidden');
        
        addLog(`✓ Message decrypted successfully with ${selectedEncryption}`, 'success');
      } else {
        document.getElementById('decrypt-error').classList.remove('hidden');
        document.getElementById('decrypt-success').classList.add('hidden');
        
        addLog(`✗ Decryption failed - incorrect parameters`, 'error');
      }
    }

    function addLog(message, type = 'info') {
      const container = document.getElementById('log-container');
      const entry = document.createElement('div');
      entry.className = 'log-entry';
      
      const timestamp = new Date().toLocaleTimeString();
      const colors = {
        info: 'text-slate-400',
        success: 'text-green-400',
        warning: 'text-amber-400',
        error: 'text-red-400',
        phase: 'text-cyan-400'
      };
      
      entry.innerHTML = `<span class="text-slate-600">[${timestamp}]</span> <span class="${colors[type]}">${message}</span>`;
      container.appendChild(entry);
      container.scrollTop = container.scrollHeight;
    }

    function updateStep(stepId, completed) {
      const step = document.getElementById(stepId);
      if (completed) {
        step.classList.remove('text-slate-500');
        step.classList.add('text-green-400');
        step.querySelector('span').classList.remove('bg-slate-700');
        step.querySelector('span').classList.add('bg-green-500');
      }
    }

    function encryptMessage(plaintext) {
      // Simulate encryption using base64 encoding
      const encoder = new TextEncoder();
      const data = encoder.encode(plaintext);
      const base64 = btoa(String.fromCharCode(...data));
      
      // Add some random characters to simulate encryption
      let encrypted = '';
      for (let i = 0; i < base64.length; i++) {
        encrypted += base64[i];
        if (Math.random() > 0.7) {
          encrypted += String.fromCharCode(65 + Math.floor(Math.random() * 26));
        }
      }
      return encrypted;
    }

    function addChatMessage(ciphertext, sender = 'client', plaintext = null) {
      // Store message in history with plaintext for decryption
      messageHistory.push({ ciphertext, sender, timestamp: Date.now(), plaintext });
      
      const chatContainer = document.getElementById('chat-messages');
      const msgDiv = document.createElement('div');
      msgDiv.className = `flex ${sender === 'client' ? 'justify-end' : 'justify-start'} animate-fadeIn flex-col gap-2`;
      
      const timestamp = new Date().toLocaleTimeString('vi-VN', { hour: '2-digit', minute: '2-digit' });
      const msgId = `msg-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
      
      msgDiv.innerHTML = `
        <div class="${sender === 'client' ? 'bg-purple-500/20 text-purple-300 self-end' : 'bg-blue-500/20 text-blue-300 self-start'} rounded-lg px-3 py-2 max-w-[80%]">
          <div class="text-xs opacity-60 mb-1 flex items-center gap-1">
            <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
            </svg>
            ${sender === 'client' ? 'Client' : 'VPN Gateway'} • ${timestamp}
          </div>
          <div class="text-xs mono break-all mb-2">${ciphertext}</div>
          <div class="text-[10px] opacity-50 border-t border-white/10 pt-2 space-y-0.5">
            <div class="flex items-center gap-1">
              <span class="text-green-400">🔐</span>
              <span>${currentEncryptionParams.encryption || 'AES-256-CBC'}</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="text-blue-400">#</span>
              <span>${(currentEncryptionParams.hash || 'sha256').toUpperCase()}</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="text-purple-400">🔑</span>
              <span>DH Group ${currentEncryptionParams.dhgroup || '14'}</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="text-amber-400">✓</span>
              <span>${currentEncryptionParams.auth === 'psk' ? 'Pre-Shared Key' : 'X.509 Certificate'}</span>
            </div>
          </div>
        </div>
        <div class="${sender === 'client' ? 'self-end' : 'self-start'}">
          <button onclick="showDecryptModal('${msgId}', '${sender}')" class="text-xs bg-amber-500/20 hover:bg-amber-500/30 text-amber-400 px-3 py-1.5 rounded-lg transition-all flex items-center gap-1.5 border border-amber-500/30">
            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 11V7a4 4 0 118 0m-4 8v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2z"/>
            </svg>
            Giải mã
          </button>
        </div>
      `;
      
      // Store message data for decryption
      msgDiv.dataset.msgId = msgId;
      msgDiv.dataset.plaintext = plaintext || '';
      
      chatContainer.appendChild(msgDiv);
      chatContainer.scrollTop = chatContainer.scrollHeight;
    }

    function addDecryptedMessage(plaintext, sender = 'client') {
      // Store decrypted message in archive
      decryptedArchive.push({ plaintext, sender, timestamp: Date.now() });
      
      const archiveContainer = document.getElementById('decrypted-archive');
      
      // Remove "no messages" placeholder
      if (archiveContainer.querySelector('.text-center')) {
        archiveContainer.innerHTML = '';
      }
      
      const msgDiv = document.createElement('div');
      msgDiv.className = 'animate-fadeIn bg-slate-800/50 rounded-lg p-3 border border-orange-500/20';
      
      const timestamp = new Date().toLocaleTimeString('vi-VN', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
      
      msgDiv.innerHTML = `
        <div class="flex items-center justify-between mb-2">
          <div class="text-xs text-slate-400 flex items-center gap-2">
            <span class="w-2 h-2 rounded-full ${sender === 'client' ? 'bg-green-400' : 'bg-blue-400'}"></span>
            ${sender === 'client' ? 'Client → Gateway' : 'Gateway → Client'}
          </div>
          <div class="text-xs text-slate-500">${timestamp}</div>
        </div>
        <div class="text-sm text-white">${plaintext}</div>
      `;
      
      archiveContainer.appendChild(msgDiv);
      archiveContainer.scrollTop = archiveContainer.scrollHeight;
    }

    function restoreMessagesForUser() {
      const chatContainer = document.getElementById('chat-messages');
      const archiveContainer = document.getElementById('decrypted-archive');
      
      // Clear current display
      chatContainer.innerHTML = '';
      archiveContainer.innerHTML = '';
      
      // BOTH users see ALL encrypted messages in chat history
      if (messageHistory.length === 0) {
        chatContainer.innerHTML = '<div class="text-center text-green-400 text-xs">✓ Kết nối bảo mật đã được thiết lập</div>';
      } else {
        messageHistory.forEach(msg => {
          const msgDiv = document.createElement('div');
          const msgId = `msg-${msg.timestamp}-${Math.random().toString(36).substr(2, 9)}`;
          msgDiv.className = `flex ${msg.sender === 'client' ? 'justify-end' : 'justify-start'} flex-col gap-2`;
          msgDiv.dataset.msgId = msgId;
          msgDiv.dataset.plaintext = msg.plaintext || '';
          
          const timestamp = new Date(msg.timestamp).toLocaleTimeString('vi-VN', { hour: '2-digit', minute: '2-digit' });
          
          msgDiv.innerHTML = `
            <div class="${msg.sender === 'client' ? 'bg-purple-500/20 text-purple-300 self-end' : 'bg-blue-500/20 text-blue-300 self-start'} rounded-lg px-3 py-2 max-w-[80%]">
              <div class="text-xs opacity-60 mb-1 flex items-center gap-1">
                <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
                </svg>
                ${msg.sender === 'client' ? 'Client' : 'VPN Gateway'} • ${timestamp}
              </div>
              <div class="text-xs mono break-all mb-2">${msg.ciphertext}</div>
              <div class="text-[10px] opacity-50 border-t border-white/10 pt-2 space-y-0.5">
                <div class="flex items-center gap-1">
                  <span class="text-green-400">🔐</span>
                  <span>${currentEncryptionParams.encryption || 'AES-256-CBC'}</span>
                </div>
                <div class="flex items-center gap-1">
                  <span class="text-blue-400">#</span>
                  <span>${(currentEncryptionParams.hash || 'sha256').toUpperCase()}</span>
                </div>
                <div class="flex items-center gap-1">
                  <span class="text-purple-400">🔑</span>
                  <span>DH Group ${currentEncryptionParams.dhgroup || '14'}</span>
                </div>
                <div class="flex items-center gap-1">
                  <span class="text-amber-400">✓</span>
                  <span>${currentEncryptionParams.auth === 'psk' ? 'Pre-Shared Key' : 'X.509 Certificate'}</span>
                </div>
              </div>
            </div>
            <div class="${msg.sender === 'client' ? 'self-end' : 'self-start'}">
              <button onclick="showDecryptModal('${msgId}', '${msg.sender}')" class="text-xs bg-amber-500/20 hover:bg-amber-500/30 text-amber-400 px-3 py-1.5 rounded-lg transition-all flex items-center gap-1.5 border border-amber-500/30">
                <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 11V7a4 4 0 118 0m-4 8v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2z"/>
                </svg>
                Giải mã
              </button>
            </div>
          `;
          
          chatContainer.appendChild(msgDiv);
        });
      }
      
      // VPN Gateway sees decrypted archive
      if (currentUser.isVPN) {
        if (decryptedArchive.length === 0) {
          archiveContainer.innerHTML = '<div class="text-center text-slate-500 text-xs">Không có tin nhắn đã giải mã</div>';
        } else {
          decryptedArchive.forEach(msg => {
            const msgDiv = document.createElement('div');
            msgDiv.className = 'bg-slate-800/50 rounded-lg p-3 border border-orange-500/20';
            
            const timestamp = new Date(msg.timestamp).toLocaleTimeString('vi-VN', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
            
            msgDiv.innerHTML = `
              <div class="flex items-center justify-between mb-2">
                <div class="text-xs text-slate-400 flex items-center gap-2">
                  <span class="w-2 h-2 rounded-full ${msg.sender === 'client' ? 'bg-green-400' : 'bg-blue-400'}"></span>
                  ${msg.sender === 'client' ? 'Client → Gateway' : 'Gateway → Client'}
                </div>
                <div class="text-xs text-slate-500">${timestamp}</div>
              </div>
              <div class="text-sm text-white">${msg.plaintext}</div>
            `;
            
            archiveContainer.appendChild(msgDiv);
          });
        }
      }
      
      chatContainer.scrollTop = chatContainer.scrollHeight;
      archiveContainer.scrollTop = archiveContainer.scrollHeight;
    }

    async function sendVPNReply(event) {
      event.preventDefault();
      
      if (!isConnected || !currentUser || !currentUser.isVPN) {
        return;
      }
      
      const input = document.getElementById('vpn-reply-input');
      const response = input.value.trim();
      
      if (!response) return;
      
      // Show server is encrypting response
      document.getElementById('plaintext-packet').textContent = response;
      await delay(200);
      
      const encryptedResponse = encryptMessage(response);
      document.getElementById('encrypted-packet').textContent = encryptedResponse;
      
      // Update stats for response
      packetCount++;
      totalBytes += new Blob([response]).size;
      document.getElementById('packet-count').textContent = packetCount;
      document.getElementById('bytes-sent').textContent = totalBytes >= 1024 
        ? (totalBytes / 1024).toFixed(1) + ' KB' 
        : totalBytes + ' B';
      
      // Add encrypted response to chat (visible to both users) - STORED AS CIPHERTEXT WITH PLAINTEXT
      addChatMessage(encryptedResponse, 'server', response);
      
      // Add decrypted response to archive (store plaintext separately for VPN to see)
      addDecryptedMessage(response, 'server');
      
      addLog(`← VPN Gateway custom reply sent (encrypted)`, 'success');
      
      input.value = '';
    }

    async function sendMessage(event) {
      event.preventDefault();
      
      if (!isConnected) {
        return;
      }
      
      const input = document.getElementById('message-input');
      const message = input.value.trim();
      
      if (!message) return;
      
      // Show plaintext
      document.getElementById('plaintext-packet').textContent = message;
      
      // Encrypt and show
      const encrypted = encryptMessage(message);
      await delay(200);
      document.getElementById('encrypted-packet').textContent = encrypted;
      
      // Update stats
      packetCount++;
      totalBytes += new Blob([message]).size;
      document.getElementById('packet-count').textContent = packetCount;
      document.getElementById('bytes-sent').textContent = totalBytes >= 1024 
        ? (totalBytes / 1024).toFixed(1) + ' KB' 
        : totalBytes + ' B';
      
      // Add encrypted message with plaintext for decryption
      addChatMessage(encrypted, 'client', message);
      
      // Add decrypted message to VPN Gateway archive storage
      addDecryptedMessage(message, 'client');
      
      addLog(`→ Encrypted message sent (${encrypted.length} bytes)`, 'info');
      
      input.value = '';
    }

    async function startVPNConnection() {
      if (isConnected) return;
      
      const connectBtn = document.getElementById('connect-btn');
      const disconnectBtn = document.getElementById('disconnect-btn');
      connectBtn.disabled = true;
      connectBtn.innerHTML = '<span class="animate-spin">⏳</span> Connecting...';
      
      const encryption = document.getElementById('encryption').value;
      const hash = document.getElementById('hash').value;
      const dhgroup = document.getElementById('dhgroup').value;
      const auth = document.getElementById('auth').value;
      
      // Store encryption parameters for decryption validation
      currentEncryptionParams = {
        encryption: encryption,
        hash: hash,
        dhgroup: dhgroup,
        auth: auth
      };
      
      // Update cipher display
      document.getElementById('current-cipher').textContent = encryption;
      document.getElementById('tunnel-cipher-display').textContent = encryption;
      
      addLog('Initializing VPN connection...', 'info');
      await delay(500);
      
      // Phase 1: IKE
      addLog('=== Phase 1: IKE Main Mode ===', 'phase');
      document.getElementById('phase1-indicator').classList.add('phase-active');
      
      await delay(800);
      addLog(`→ Sending SA proposal (${encryption}, ${hash.toUpperCase()}, DH Group ${dhgroup})`, 'info');
      updateStep('ike-step1', true);
      
      await delay(800);
      addLog('← Received SA acceptance from gateway', 'success');
      addLog(`→ Initiating Diffie-Hellman key exchange (Group ${dhgroup})`, 'info');
      updateStep('ike-step2', true);
      
      await delay(1000);
      addLog('← DH public value received', 'success');
      addLog('Generating shared secret...', 'info');
      
      await delay(800);
      if (auth === 'psk') {
        addLog('→ Authenticating with Pre-Shared Key', 'info');
      } else {
        addLog('→ Authenticating with X.509 Certificate', 'info');
      }
      updateStep('ike-step3', true);
      
      await delay(800);
      addLog('← Authentication successful', 'success');
      addLog('✓ IKE SA established', 'success');
      updateStep('ike-step4', true);
      
      document.getElementById('phase1-indicator').classList.remove('phase-active');
      document.getElementById('phase1-indicator').classList.add('phase-complete');
      
      // Phase 2: IPSEC
      await delay(500);
      addLog('=== Phase 2: IPSEC Quick Mode ===', 'phase');
      document.getElementById('phase2-indicator').classList.add('phase-active');
      
      await delay(800);
      addLog('→ Initiating Quick Mode negotiation', 'info');
      updateStep('ipsec-step1', true);
      
      await delay(800);
      addLog(`→ Proposing IPSEC SA (ESP, ${encryption}, ${hash.toUpperCase()})`, 'info');
      addLog('← IPSEC SA parameters accepted', 'success');
      updateStep('ipsec-step2', true);
      
      await delay(800);
      addLog('Generating session keys from SKEYID...', 'info');
      addLog('✓ Session keys generated successfully', 'success');
      updateStep('ipsec-step3', true);
      
      await delay(800);
      addLog('✓ IPSEC tunnel established!', 'success');
      updateStep('ipsec-step4', true);
      
      document.getElementById('phase2-indicator').classList.remove('phase-active');
      document.getElementById('phase2-indicator').classList.add('phase-complete');
      
      // Connection established
      isConnected = true;
      
      document.getElementById('client-node').classList.add('connected');
      document.getElementById('server-node').classList.add('connected');
      document.getElementById('client-status').textContent = 'Connected';
      document.getElementById('client-status').classList.remove('bg-slate-700', 'text-slate-300');
      document.getElementById('client-status').classList.add('bg-green-500/20', 'text-green-400');
      document.getElementById('server-status').textContent = 'Connected';
      document.getElementById('server-status').classList.remove('bg-slate-700', 'text-slate-300');
      document.getElementById('server-status').classList.add('bg-green-500/20', 'text-green-400');
      
      document.getElementById('tunnel-flow').classList.add('opacity-100');
      
      connectBtn.classList.add('hidden');
      disconnectBtn.classList.remove('hidden');
      
      // Enable messaging
      document.getElementById('message-input').disabled = false;
      document.getElementById('send-btn').disabled = false;
      
      // Restore messages for current user
      restoreMessagesForUser();
      
      // Enable VPN reply if VPN user
      if (currentUser && currentUser.isVPN) {
        document.getElementById('vpn-reply-input').disabled = false;
        document.getElementById('vpn-reply-btn').disabled = false;
      }
      
      addLog('���� Secure tunnel active - All traffic encrypted', 'success');
      addLog('💬 Messaging interface enabled', 'success');
      
      // Start packet animation
      startPacketAnimation();
    }

    function startPacketAnimation() {
      const container = document.getElementById('tunnel-packets');
      connectionInterval = setInterval(() => {
        if (!isConnected) return;
        
        const packet = document.createElement('div');
        packet.className = 'w-2 h-2 rounded-full bg-green-400 data-packet';
        packet.style.animationDuration = `${1 + Math.random()}s`;
        container.appendChild(packet);
        
        setTimeout(() => packet.remove(), 2000);
      }, 500);
    }

    function disconnectVPN() {
      if (!isConnected) return;
      
      isConnected = false;
      clearInterval(connectionInterval);
      
      addLog('Disconnecting VPN...', 'warning');
      addLog('→ Sending DELETE notification', 'info');
      
      setTimeout(() => {
        addLog('← Acknowledgment received', 'info');
        addLog('Tunnel closed', 'warning');
        
        // Reset UI
        document.getElementById('client-node').classList.remove('connected');
        document.getElementById('server-node').classList.remove('connected');
        document.getElementById('client-status').textContent = 'Disconnected';
        document.getElementById('client-status').classList.remove('bg-green-500/20', 'text-green-400');
        document.getElementById('client-status').classList.add('bg-slate-700', 'text-slate-300');
        document.getElementById('server-status').textContent = 'Listening';
        document.getElementById('server-status').classList.remove('bg-green-500/20', 'text-green-400');
        document.getElementById('server-status').classList.add('bg-slate-700', 'text-slate-300');
        
        document.getElementById('tunnel-flow').classList.remove('opacity-100');
        document.getElementById('tunnel-packets').innerHTML = '';
        
        // Disable messaging
        document.getElementById('message-input').disabled = true;
        document.getElementById('message-input').value = '';
        document.getElementById('send-btn').disabled = true;
        document.getElementById('chat-messages').innerHTML = '<div class="text-center text-slate-500 text-xs">Kết nối VPN để gửi tin nhắn bảo mật</div>';
        
        // Disable VPN reply
        document.getElementById('vpn-reply-input').disabled = true;
        document.getElementById('vpn-reply-input').value = '';
        document.getElementById('vpn-reply-btn').disabled = true;
        
        // Reset encryption display
        document.getElementById('plaintext-packet').textContent = 'Waiting for message...';
        document.getElementById('encrypted-packet').textContent = 'Waiting for message...';
        
        // Clear decrypted archive
        document.getElementById('decrypted-archive').innerHTML = '<div class="text-center text-slate-500 text-xs">Không có tin nh����n đã giải mã</div>';
        
        // Reset phase indicators
        ['phase1-indicator', 'phase2-indicator'].forEach(id => {
          const el = document.getElementById(id);
          el.classList.remove('phase-complete', 'phase-active');
        });
        
        // Reset steps
        ['ike-step1', 'ike-step2', 'ike-step3', 'ike-step4', 'ipsec-step1', 'ipsec-step2', 'ipsec-step3', 'ipsec-step4'].forEach(id => {
          const step = document.getElementById(id);
          step.classList.remove('text-green-400');
          step.classList.add('text-slate-500');
          step.querySelector('span').classList.remove('bg-green-500');
          step.querySelector('span').classList.add('bg-slate-700');
        });
        
        const connectBtn = document.getElementById('connect-btn');
        const disconnectBtn = document.getElementById('disconnect-btn');
        connectBtn.classList.remove('hidden');
        connectBtn.disabled = false;
        connectBtn.innerHTML = `
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"/>
          </svg>
          <span>Connect VPN</span>
        `;
        disconnectBtn.classList.add('hidden');
      }, 500);
    }

    function delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    }

    async function onConfigChange(config) {
      const mainTitle = document.getElementById('main-title');
      const subtitle = document.getElementById('subtitle');
      
      if (mainTitle) mainTitle.textContent = config.main_title || defaultConfig.main_title;
      if (subtitle) subtitle.textContent = config.subtitle || defaultConfig.subtitle;
      
      const customFont = config.font_family || defaultConfig.font_family;
      document.body.style.fontFamily = `${customFont}, sans-serif`;
    }

    function mapToCapabilities(config) {
      return {
        recolorables: [
          {
            get: () => config.background_color || defaultConfig.background_color,
            set: (value) => {
              config.background_color = value;
              window.elementSdk.setConfig({ background_color: value });
            }
          },
          {
            get: () => config.primary_color || defaultConfig.primary_color,
            set: (value) => {
              config.primary_color = value;
              window.elementSdk.setConfig({ primary_color: value });
            }
          }
        ],
        borderables: [],
        fontEditable: {
          get: () => config.font_family || defaultConfig.font_family,
          set: (value) => {
            config.font_family = value;
            window.elementSdk.setConfig({ font_family: value });
          }
        },
        fontSizeable: undefined
      };
    }

    function mapToEditPanelValues(config) {
      return new Map([
        ['main_title', config.main_title || defaultConfig.main_title],
        ['subtitle', config.subtitle || defaultConfig.subtitle]
      ]);
    }

    // Initialize SDK
    if (window.elementSdk) {
      window.elementSdk.init({
        defaultConfig,
        onConfigChange,
        mapToCapabilities,
        mapToEditPanelValues
      });
    }
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9bdb9b7f968cacf8',t:'MTc2ODM3NzcwNy4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
