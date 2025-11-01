# Bank
Bank
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Robux Wallet (로컬 저장소 - 가상 송금)</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&family=Noto+Sans+KR:wght@300;400;500;700;800;900&display=swap');
        /* 메인 액센트 컬러 */
        :root {
            --accent-color: #A855F7; /* Violet-500 (보라색 계열) */
        }

        /* 뱅크 앱 스타일의 클린 다크 모드 */
        body {
            font-family: 'Inter', 'Noto Sans KR', sans-serif;
            background-color: #0A0A0A; /* 가장 어두운 배경 */
            color: #FFFFFF;
        }
        .app-container {
            max-width: 800px; /* 태블릿 크기로 확장 */
            margin: 0 auto;
            min-height: 100vh;
            background-color: #0A0A0A;
        }
        .card {
            background-color: #1B1B1B; /* UI 요소 배경 */
            border-radius: 30px; /* 슈퍼 라운드 */
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
        }
        /* 입력 필드 스타일 통일 */
        .app-input {
            width: 100%;
            padding: 1rem 1.25rem;
            border: 1px solid #333333;
            border-radius: 18px;
            background-color: #2A2A2A;
            color: #FFFFFF;
            font-size: 1.125rem;
            transition: border-color 0.2s, box-shadow 0.2s;
        }
        .app-input:focus {
            border-color: var(--accent-color); /* 보라색 하이라이트 */
            outline: none;
            box-shadow: 0 0 0 2px var(--accent-color);
        }
        
        /* PIN 입력 필드 */
        .pin-input {
            width: 55px;
            height: 65px; 
            font-size: 2rem;
            text-align: center;
            border: 2px solid #333;
            border-radius: 15px;
            background-color: #2A2A2A;
            color: #FFFFFF;
            font-family: 'Inter', 'Noto Sans KR', sans-serif;
        }
        /* type="password"에 대한 추가 스타일링 */
        input[type="password"].pin-input {
            line-height: 65px;
            font-size: 2rem; 
            -webkit-text-security: disc; 
        }

        .pin-input:focus {
            border-color: var(--accent-color);
            box-shadow: 0 0 0 3px rgba(168, 85, 247, 0.5); /* 보라색 그림자 */
            outline: none;
        }
        
        /* 잔액 입력 필드 스타일 */
        #robux-balance-input {
            border: none;
            background: transparent;
            font-size: 3.5rem; 
            font-weight: 900;
            text-align: right;
            padding: 0;
            line-height: 1;
        }
        #robux-balance-input:focus {
            outline: none;
            box-shadow: none;
        }
        
        /* PIN 오류 시 좌우 흔들림 애니메이션 */
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            20%, 60% { transform: translateX(-8px); }
            40%, 80% { transform: translateX(8px); }
        }
        .shake-error {
            animation: shake 0.5s cubic-bezier(.36,.07,.19,.97) both;
            transform: translate3d(0, 0, 0);
            backface-visibility: hidden;
            perspective: 1000px;
        }

        /* 사이드바 */
        #sidebar {
            width: 280px;
            right: -280px;
            transition: right 0.3s cubic-bezier(0.4, 0.0, 0.2, 1);
            background-color: #1B1B1B;
            box-shadow: -6px 0 15px rgba(0, 0, 0, 0.6);
        }
        #sidebar.open {
            right: 0;
        }
        
        /* 모달 애니메이션 CSS (Fade-in/out & Scale) */
        .modal-animated {
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease-out;
        }
        .modal-animated.show {
            opacity: 1;
            pointer-events: auto;
        }
        
        .modal-content-animated {
            transform: scale(0.95);
            transition: transform 0.3s cubic-bezier(0.25, 0.8, 0.5, 1.25), opacity 0.3s ease-out;
        }
        .modal-animated.show .modal-content-animated {
            transform: scale(1);
        }

        /* --- 성공 애니메이션 수정 완료 --- */
        /* 성공 애니메이션의 체크마크 그리기 */
        @keyframes draw-check {
            0% { stroke-dashoffset: 1000; }
            100% { stroke-dashoffset: 0; }
        }
        
        .success-check {
            stroke-dasharray: 1000;
            stroke-dashoffset: 1000;
        }

        /* 성공 애니메이션 원형의 크기 조정 애니메이션 */
        @keyframes scale-up {
            0% { transform: scale(0.9); opacity: 0; }
            100% { transform: scale(1); opacity: 1; }
        }
        
        #success-animation {
            animation: scale-up 0.3s ease-out;
        }
        /* ---------------------------------- */

    </style>
</head>
<body>

    <div id="app" class="app-container p-5 flex flex-col relative overflow-hidden">
        
        <div id="login-overlay" class="fixed inset-0 bg-[#0A0A0A] z-[70] flex flex-col items-center justify-center p-8 transition-opacity duration-500">
            <div class="w-full max-w-xs text-center">
                <h2 id="login-title" class="text-3xl font-extrabold text-[--accent-color] mb-6">로딩 중...</h2>
    
                <div id="pin-input-container" class="flex justify-center space-x-3 mb-8">
                    </div>
                <button id="auth-button" class="w-full py-4 bg-gray-700 text-gray-400 font-bold rounded-2xl transition duration-150 active:scale-[0.98] cursor-not-allowed text-lg" disabled>
                    확인
                </button>
                <p id="auth-message" class="mt-4 text-sm text-gray-400 font-light"></p>
                <p class="mt-8 text-xs text-gray-500 font-light">
                    *데이터 보안 및 사용자 구분을 위해 PIN이 필요합니다.
                </p>
            </div>
        </div>

        <div id="main-content" class="w-full opacity-0 transition-opacity duration-500 pointer-events-none z-10">
            
            <header class="w-full mb-8 flex justify-between items-center h-10">
                <h1 class="text-3xl font-extrabold text-gray-50">Robux Wallet</h1>
                <button id="menu-toggle-button" class="p-3 rounded-full bg-[#1B1B1B] hover:bg-[#2A2A2A] active:scale-90 transition duration-150 shadow-lg">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-[--accent-color]"><line x1="4" x2="20" y1="12" y2="12"/><line x1="4" x2="20" y1="6" y2="6"/><line x1="4" x2="20" y1="18" y2="18"/></svg>
                </button>
            </header>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">

                <div class="card p-6 md:col-span-1">
                    <div class="flex items-center justify-between mb-4">
                        <h2 class="text-xl font-medium text-gray-300">총 자산 (Robux)</h2>
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-[--accent-color]"><path d="M19 7V4a1 1 0 0 0-1-1H5a2 2 0 0 0 0 4h15a1 1 0 0 1 1 1v4h-3a2 2 0 0 0 0 4h3v2a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1V3"/></svg>
                    </div>
                    
                    <div class="flex flex-col space-y-4">
                        <div class="flex items-end justify-end">
                            <input
                                type="number"
                                id="robux-balance-input"
                                class="text-5xl font-extrabold text-right border-none bg-transparent focus:ring-0 p-0 w-3/4"
                                value="0"
                                placeholder="0"
                                style="line-height: 1;"
                            >
                            <span class="text-3xl font-bold text-gray-400 ml-2 mb-1">RBX</span>
                        </div>
                        
                        <p class="text-xs text-right text-gray-500">
                            *잔액 입력 후 필드를 벗어나면(Blur) 자동 저장됩니다.
                        </p>

                        <div class="text-2xl font-semibold text-gray-400 text-right border-t border-gray-700 pt-3">
                            ≈ <span id="krw-balance" class="text-[--accent-color] font-bold">0</span> 원 (KRW)
                        </div>
                    </div>
                </div>

                <div class="card p-6 md:col-span-1">
                    <h2 class="text-xl font-medium text-gray-300 mb-4 flex items-center">
                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mr-2 text-red-400"><path d="m22 17-7-7-4 4-8-8"/><path d="M16 22h6V16"/></svg>
                        지출 관리 및 피드백
                    </h2>
                    
                    <div class="mb-5">
                        <label for="last-month-spending" class="block text-sm font-light text-gray-400 mb-2">지난달 지출 (RBX) 기록</label>
                        <div class="flex items-end justify-between bg-[#2A2A2A] rounded-2xl p-3 border border-[#333333] focus-within:border-[--accent-color] focus-within:ring-2 focus-within:ring-[--accent-color]">
                            <input
                                type="number"
                                id="last-month-spending"
                                placeholder="0"
                                class="w-full text-xl font-bold bg-transparent border-none p-0 focus:ring-0 text-right"
                                value="0"
                            >
                            <span class="text-lg font-bold text-gray-400 ml-2">RBX</span>
                        </div>
                    </div>

                    <div id="feedback-result" class="bg-[#2A2A2A] p-4 rounded-2xl border border-gray-700">
                        <p class="text-sm font-light text-gray-400 mb-1">다음 달 권장 지출 예산</p>
                        <div class="text-3xl font-extrabold text-[--accent-color]">
                            <span id="suggested-budget">0</span> RBX
                        </div>
                        <p id="budget-message" class="mt-2 text-xs text-gray-400 font-light">지출을 입력하고 [저장] 버튼을 눌러보세요.</p>
                    </div>

                    <button id="save-data-button" class="mt-6 w-full py-4 bg-[--accent-color] text-[#0A0A0A] font-extrabold rounded-2xl hover:bg-violet-600 transition duration-150 active:scale-[0.98] text-lg shadow-xl shadow-violet-500/30">
                        지출 데이터 저장 및 예산 업데이트
                    </button>
                </div>
            </div>

            <div class="w-full mt-6">
                 <button id="open-transfer-modal-main" class="w-full text-center py-4 px-4 bg-[#1B1B1B] text-gray-200 font-bold rounded-2xl border border-violet-500/50 hover:bg-[#2A2A2A] transition flex items-center justify-center active:scale-[0.99] shadow-lg">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mr-3 text-red-400"><path d="m22 2-7 20-4-9-9-4Z"/><path d="M22 2 11 13"/></svg>
                    Robux 송금하기
                </button>
            </div>
            
        </div>
        
        <div id="sidebar-backdrop" class="fixed inset-0 bg-black bg-opacity-70 z-[40] hidden" onclick="toggleSidebar()"></div>
        <div id="sidebar" class="fixed top-0 bottom-0 h-full z-[50] p-6 flex flex-col space-y-4 rounded-l-3xl">
            <h3 class="text-2xl font-bold text-gray-50 mb-4 border-b border-gray-700 pb-3">설정 및 기능</h3>
            
            <button id="open-transfer-modal-sidebar" class="w-full text-left py-3 px-4 bg-[#2A2A2A] text-gray-50 font-medium rounded-xl flex items-center hover:bg-[#383838] transition active:scale-[0.99]">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mr-3 text-red-400"><path d="m22 2-7 20-4-9-9-4Z"/><path d="M22 2 11 13"/></svg>
                로벅스 송금하기
            </button>

            <div class="flex-grow"></div>
            
            <p id="user-id-display" class="text-xs text-gray-500 font-light text-left pt-4 border-t border-gray-700 truncate">사용자 ID: 로컬 저장소</p>
            
            <button id="sidebar-logout-button" class="w-full py-3 bg-red-600 text-white font-medium rounded-xl text-base hover:bg-red-700 transition active:scale-[0.98]">
                잠금 화면으로 돌아가기
            </button>
        </div>

        <div id="transfer-modal" class="fixed inset-0 bg-black bg-opacity-80 z-[60] flex items-center justify-center modal-animated">
             <div class="card p-7 w-full max-w-sm m-4 modal-content-animated">
                <h3 class="text-3xl font-extrabold text-[--accent-color] mb-1 text-center">Robux 송금</h3>
                <p class="text-xs text-gray-400 text-center mb-6 border-b border-gray-700 pb-4">
                    (가상 송금: 실제 서버 전송 없이 잔액만 차감됩니다.)
                </p>

                <div class="space-y-4 mb-6">
                    <div>
                        <label for="recipient-id" class="block text-sm font-light text-gray-400 mb-2">받는 사람 ID (가상)</label>
                        <input type="text" id="recipient-id" class="app-input" placeholder="Roblox ID 또는 닉네임 (ex: VirtualUser)" value="VirtualUser">
                    </div>
                    <div>
                        <label for="transfer-amount" class="block text-sm font-light text-gray-400 mb-2">송금 금액 (RBX)</label>
                        <div class="flex items-center">
                            <input type="number" id="transfer-amount" class="app-input text-right flex-grow" placeholder="0">
                            <span class="text-xl font-bold text-gray-400 ml-3">RBX</span>
                        </div>
                    </div>
                </div>

                <p id="transfer-message" class="text-sm text-red-400 mb-4 font-light text-center hidden"></p>

                <div class="flex space-x-3">
                    <button id="cancel-transfer-button" class="w-1/3 py-3 text-gray-400 font-medium rounded-2xl bg-[#2A2A2A] hover:bg-[#383838] transition active:scale-[0.98]">
                        취소
                    </button>
                    <button id="confirm-transfer-button" class="w-2/3 py-3 bg-red-600 text-white font-extrabold rounded-2xl hover:bg-red-700 transition active:scale-[0.98]">
                        송금 실행
                    </button>
                </div>
             </div>
        </div>

        <div id="success-modal" class="fixed inset-0 bg-black bg-opacity-80 z-[80] flex items-center justify-center modal-animated">
            <div id="success-animation" class="flex flex-col items-center p-8 card w-64 h-64 justify-center modal-content-animated">
                <svg width="80" height="80" viewBox="0 0 100 100" class="success-circle mb-4">
                    <circle cx="50" cy="50" r="45" fill="#C4B5FD" /> 
                    <path class="success-check" fill="none" stroke="#1B1B1B" stroke-width="8" stroke-linecap="round" stroke-linejoin="round" d="M30 50 L45 65 L70 40"/>
                </svg>
                <p class="text-xl font-bold text-white mt-4">저장 완료!</p>
            </div>
        </div>


        <div id="message-box" class="fixed bottom-0 w-full p-4 transition-transform duration-300 transform translate-y-full z-[30]">
            <div id="message-content" class="bg-red-600 text-white p-3 rounded-xl shadow-lg font-medium text-center max-w-sm mx-auto"></div>
        </div>
    </div>

    <script>
        // =========================================================================
        // 1. 전역 설정 및 변수
        // =========================================================================
        const ROBEX_TO_KRW_RATE = 12.5;
        const PIN_LENGTH = 4;
        const INITIAL_ROBUX = 500000;
        
        // 로컬 저장소 키
        const KEY_PIN = 'local_wallet_pin';
        const KEY_BALANCE = 'local_wallet_balance';
        const KEY_SPENDING = 'local_wallet_spending';

        let currentPin = null; 
        let isPinSet = false;
        let currentRobuxBalance = 0;

        // UI 요소
        const mainContent = document.getElementById('main-content');
        const loginOverlay = document.getElementById('login-overlay');
        const loginTitle = document.getElementById('login-title');
        const pinInputContainer = document.getElementById('pin-input-container');
        const authButton = document.getElementById('auth-button');
        const authMessage = document.getElementById('auth-message');
        const sidebar = document.getElementById('sidebar');
        const sidebarBackdrop = document.getElementById('sidebar-backdrop');
        const menuToggleButton = document.getElementById('menu-toggle-button');
        const sidebarLogoutButton = document.getElementById('sidebar-logout-button');
        const openTransferModalButtonMain = document.getElementById('open-transfer-modal-main');
        const openTransferModalButtonSidebar = document.getElementById('open-transfer-modal-sidebar');
        const transferModal = document.getElementById('transfer-modal');
        const transferModalContent = transferModal.querySelector('.modal-content-animated');
        const cancelTransferButton = document.getElementById('cancel-transfer-button');
        const successModal = document.getElementById('success-modal');
        const successModalContent = successModal.querySelector('.modal-content-animated');

        const robuxBalanceInput = document.getElementById('robux-balance-input');
        const krwBalanceDisplay = document.getElementById('krw-balance');
        const lastMonthSpendingInput = document.getElementById('last-month-spending');
        const suggestedBudgetDisplay = document.getElementById('suggested-budget');
        const budgetMessage = document.getElementById('budget-message');
        const saveButton = document.getElementById('save-data-button');
        const userIdDisplay = document.getElementById('user-id-display');
        const messageBox = document.getElementById('message-box');
        const messageContent = document.getElementById('message-content');
        
        // 송금 모달 내부 요소 (추가됨)
        const recipientIdInput = document.getElementById('recipient-id');
        const transferAmountInput = document.getElementById('transfer-amount');
        const confirmTransferButton = document.getElementById('confirm-transfer-button');
        const transferMessage = document.getElementById('transfer-message');


        // Tone.js Synth (싱글톤)
        let clickSynth;
        let successSynth;
        let errorSynth;
        
        // =========================================================================
        // 2. UI 및 유틸리티 함수 (사운드 포함)
        // =========================================================================
        
        const initializeAudio = () => {
             try {
                if (Tone.context.state !== 'running') {
                    Tone.start();
                }
                clickSynth = new Tone.Synth({
                    oscillator: { type: 'sine' },
                    envelope: { attack: 0.005, decay: 0.05, sustain: 0.0, release: 0.05 }
                }).toDestination();
                successSynth = new Tone.Synth({
                    oscillator: { type: 'sine' },
                    envelope: { attack: 0.005, decay: 0.1, sustain: 0.0, release: 0.1 }
                }).toDestination();
                errorSynth = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: 'sawtooth' },
                    envelope: { attack: 0.01, decay: 0.2, sustain: 0.0, release: 0.1 }
                }).toDestination();
            } catch (e) {
                console.warn("Tone.js 초기화 오류:", e);
            }
        };

        const playClickSound = () => { if (clickSynth) { try { clickSynth.triggerAttackRelease("C5", "16n"); } catch (e) { /* ignore */ } }; };
        const playSuccessSound = () => { if (successSynth) { try { successSynth.triggerAttackRelease("D5", "16n"); successSynth.triggerAttackRelease("G5", "16n", Tone.now() + 0.1); } catch (e) { /* ignore */ } }; };
        const playErrorSound = () => { if (errorSynth) { try { errorSynth.triggerAttackRelease(["C3", "C#3"], "8n"); } catch (e) { /* ignore */ } }; };


        const showMessage = (message, isError = false) => {
            messageContent.textContent = message;
            messageContent.className = isError 
                ? 'bg-red-600 text-white p-3 rounded-xl shadow-lg font-medium text-center max-w-sm mx-auto'
                : 'bg-violet-600 text-white p-3 rounded-xl shadow-lg font-medium text-center max-w-sm mx-auto';
            messageBox.classList.remove('translate-y-full');
            messageBox.classList.add('translate-y-0');

            setTimeout(() => {
                messageBox.classList.remove('translate-y-0');
                messageBox.classList.add('translate-y-full');
            }, 3000);
        };

        const formatNumber = (num) => {
            if (num === null || isNaN(num)) return '0';
            return Math.floor(num).toLocaleString('ko-KR');
        };
        
        const toggleModal = (modalElement, isOpening) => {
            if (isOpening) {
                modalElement.classList.remove('hidden');
                void modalElement.offsetWidth;
                modalElement.classList.add('show');
            } else {
                modalElement.classList.remove('show');
                setTimeout(() => {
                    modalElement.classList.add('hidden');
                }, 300);
            }
        };
        
        const toggleMainAppVisibility = (visible) => {
            if (visible) {
                loginOverlay.style.opacity = '0';
                loginOverlay.style.pointerEvents = 'none';
                setTimeout(() => {
                    mainContent.style.opacity = '1';
                    mainContent.style.pointerEvents = 'auto';
                }, 500);
            } else {
                loginOverlay.style.opacity = '1';
                loginOverlay.style.pointerEvents = 'auto';
                mainContent.style.opacity = '0';
                mainContent.style.pointerEvents = 'none';
            }
        };
        
        window.toggleSidebar = () => {
            playClickSound();
            const isOpen = sidebar.classList.toggle('open');
            sidebarBackdrop.style.display = isOpen ? 'block' : 'none';
        };

        const openTransferModal = () => {
            playClickSound();
            if (sidebar.classList.contains('open')) { toggleSidebar(); }
            
            // 모달 초기화
            transferAmountInput.value = '';
            transferMessage.classList.add('hidden');
            
            toggleModal(transferModal, true);
        };
        
        const closeTransferModal = () => { playClickSound(); toggleModal(transferModal, false); };

        const showSuccessAnimation = (message = '저장 완료!') => {
            playSuccessSound();
            
            const checkPath = successModalContent.querySelector('.success-check');
            // 애니메이션 초기화 및 재시작
            checkPath.style.animation = 'none'; 
            void checkPath.offsetWidth; 
            checkPath.style.animation = 'draw-check 0.6s cubic-bezier(0.65, 0, 0.45, 1) forwards'; 
            
            successModalContent.querySelector('p').textContent = message;
            toggleModal(successModal, true);
            setTimeout(() => {
                toggleModal(successModal, false);
                checkPath.style.animation = 'none'; // 애니메이션 종료 후 초기화
            }, 1500);
        };

        // =========================================================================
        // 3. PIN 및 인증 로직 (Local Storage)
        // =========================================================================

        const setupPinInputs = () => {
            pinInputContainer.innerHTML = '';
            for (let i = 0; i < PIN_LENGTH; i++) {
                const input = document.createElement('input');
                input.type = 'password';
                input.className = 'pin-input';
                input.maxLength = 1;
                input.dataset.index = i;
                pinInputContainer.appendChild(input);

                input.addEventListener('input', (e) => {
                    if (e.target.value.length === 1 && i < PIN_LENGTH - 1) {
                        pinInputContainer.children[i + 1].focus();
                    }
                    updateAuthButtonState();
                });

                input.addEventListener('keydown', (e) => {
                    if (e.key === 'Backspace' && e.target.value === '' && i > 0) {
                        pinInputContainer.children[i - 1].focus();
                    }
                });
            }
        };

        const getPinFromInputs = () => {
            return Array.from(pinInputContainer.children).map(input => input.value).join('');
        };

        const updateAuthButtonState = () => {
            const pin = getPinFromInputs();
            if (pin.length === PIN_LENGTH) {
                authButton.disabled = false;
                authButton.textContent = '확인';
                authButton.classList.remove('bg-gray-700', 'text-gray-400', 'cursor-not-allowed');
                authButton.classList.add('bg-[--accent-color]', 'text-[#0A0A0A]', 'hover:bg-violet-600');
            } else {
                authButton.disabled = true;
                authButton.textContent = '확인';
                authButton.classList.remove('bg-[--accent-color]', 'text-[#0A0A0A]', 'hover:bg-violet-600');
                authButton.classList.add('bg-gray-700', 'text-gray-400', 'cursor-not-allowed');
            }
        };

        const handleAuthAction = () => {
            playClickSound();
            const inputPin = getPinFromInputs();

            if (isPinSet) {
                // 로그인 시도
                if (inputPin === currentPin) {
                    toggleMainAppVisibility(true);
                    showMessage('로그인 성공!', false);
                } else {
                    playErrorSound();
                    authMessage.textContent = 'PIN이 일치하지 않습니다.';
                    pinInputContainer.classList.add('shake-error');
                    setTimeout(() => {
                        pinInputContainer.classList.remove('shake-error');
                        Array.from(pinInputContainer.children).forEach(input => input.value = '');
                        pinInputContainer.children[0].focus();
                        authMessage.textContent = '';
                    }, 500);
                }
            } else {
                // PIN 설정 시도
                currentPin = inputPin;
                localStorage.setItem(KEY_PIN, currentPin);
                isPinSet = true;
                loginTitle.textContent = '로그인';
                authMessage.textContent = '새로운 PIN이 설정되었습니다. 로그인합니다.';
                
                // 잔액 및 지출 데이터 초기 로드 및 설정 (PIN 설정 시 로컬 저장소 초기값 설정)
                loadLocalData();

                setTimeout(() => {
                    toggleMainAppVisibility(true);
                    showMessage('PIN 설정 및 로그인 완료!', false);
                }, 1000);
            }
        };

        const handleLogout = () => {
            playClickSound();
            toggleMainAppVisibility(false);
            loginTitle.textContent = '로그인'; // 로그아웃 후 로그인 화면으로 전환
            isPinSet = localStorage.getItem(KEY_PIN) !== null;
            authMessage.textContent = isPinSet ? 'PIN을 입력하세요.' : '새로운 PIN(4자리)을 설정하세요.';
            Array.from(pinInputContainer.children).forEach(input => input.value = '');
            updateAuthButtonState();
            pinInputContainer.children[0].focus();
        };

        // =========================================================================
        // 4. 로컬 저장소 데이터 로드 및 저장
        // =========================================================================

        const loadLocalData = () => {
            // 1. PIN 로드
            currentPin = localStorage.getItem(KEY_PIN);
            isPinSet = currentPin !== null;
            
            // PIN 설정 상태에 따라 초기 화면 결정
            if (isPinSet) {
                loginTitle.textContent = '로그인';
                authMessage.textContent = 'PIN을 입력하세요.';
            } else {
                loginTitle.textContent = 'PIN 설정';
                authMessage.textContent = '새로운 PIN(4자리)을 설정하세요.';
            }
            
            // 2. 잔액 로드
            const savedBalance = localStorage.getItem(KEY_BALANCE);
            currentRobuxBalance = savedBalance ? parseFloat(savedBalance) : INITIAL_ROBUX;
            robuxBalanceInput.value = currentRobuxBalance;
            updateKrwBalance();

            // 3. 지출 로드
            const savedSpending = localStorage.getItem(KEY_SPENDING);
            const lastSpent = savedSpending ? parseFloat(savedSpending) : 0;
            lastMonthSpendingInput.value = lastSpent;
            updateBudgetFeedback(lastSpent);
        };
        
        const saveBalanceData = (newBalance) => {
            const balanceToSave = parseFloat(newBalance) || 0;
            localStorage.setItem(KEY_BALANCE, balanceToSave.toString());
            return true;
        };

        const saveSpendingData = () => {
            playClickSound();
            saveButton.disabled = true;
            saveButton.textContent = '저장 중...';
            saveButton.classList.remove('bg-[--accent-color]', 'hover:bg-violet-600');
            saveButton.classList.add('bg-gray-700', 'cursor-not-allowed', 'shadow-none');
            
            const lastSpent = parseFloat(lastMonthSpendingInput.value) || 0;
            
            // 로컬 저장소에 저장
            localStorage.setItem(KEY_SPENDING, lastSpent.toString());

            // 피드백 업데이트
            updateBudgetFeedback(lastSpent);

            showSuccessAnimation('지출 데이터 저장 완료!');

            saveButton.disabled = false;
            saveButton.textContent = '지출 데이터 저장 및 예산 업데이트';
            saveButton.classList.remove('bg-gray-700', 'cursor-not-allowed', 'shadow-none');
            saveButton.classList.add('bg-[--accent-color]', 'hover:bg-violet-600');
        };

        // =========================================================================
        // 5. 로벅스/지출 로직 (로컬)
        // =========================================================================

        const updateKrwBalance = () => {
            const robux = parseFloat(robuxBalanceInput.value) || 0;
            currentRobuxBalance = robux; 
            const krw = robux * ROBEX_TO_KRW_RATE;
            krwBalanceDisplay.textContent = formatNumber(krw);
        };

        const handleBalanceInputSave = () => {
            const newBalance = parseFloat(robuxBalanceInput.value);
            if (isNaN(newBalance) || newBalance < 0) {
                robuxBalanceInput.value = currentRobuxBalance;
                updateKrwBalance(); 
                showMessage('유효한 잔액을 입력해주세요.', true);
                return;
            }

            if (newBalance === currentRobuxBalance) {
                updateKrwBalance();
                return;
            }
            
            // 로컬 저장소에 저장
            saveBalanceData(newBalance);
            currentRobuxBalance = newBalance;
            updateKrwBalance();
            showSuccessAnimation('잔액 저장 완료!');
        };

        const updateBudgetFeedback = (lastSpentRobux) => {
            const spent = parseFloat(lastSpentRobux) || 0;
            let suggestedBudget = 0;
            let message = '';

            if (spent > 0) {
                suggestedBudget = Math.floor(spent * 0.85);
                suggestedBudgetDisplay.textContent = formatNumber(suggestedBudget);
                message = `지난달 ${formatNumber(spent)} RBX 지출. 절약을 위해 이번 달 ${formatNumber(suggestedBudget)} RBX 이하 사용을 권장합니다.`;
            } else {
                suggestedBudget = 0;
                suggestedBudgetDisplay.textContent = 'N/A';
                message = '지출 기록이 없습니다. 기록을 남겨보세요.';
            }

            budgetMessage.textContent = message;
        };
        
        // --- 가상 송금 로직 ---
        const handleTransfer = () => {
            playClickSound();
            transferMessage.classList.add('hidden'); // 메시지 초기화
            
            const amount = parseFloat(transferAmountInput.value);
            const recipient = recipientIdInput.value.trim();

            if (!recipient) {
                playErrorSound();
                transferMessage.textContent = '받는 사람 ID를 입력해주세요.';
                transferMessage.classList.remove('hidden');
                return;
            }

            if (isNaN(amount) || amount <= 0 || amount % 1 !== 0) {
                playErrorSound();
                transferMessage.textContent = '유효한 로벅스 금액을 입력해주세요. (정수만 가능)';
                transferMessage.classList.remove('hidden');
                return;
            }

            if (amount > currentRobuxBalance) {
                playErrorSound();
                transferMessage.textContent = '잔액이 부족합니다.';
                transferMessage.classList.remove('hidden');
                return;
            }
            
            // 1. 잔액 차감 (가상 송금)
            const newBalance = currentRobuxBalance - amount;
            
            // 2. 로컬 저장소 업데이트
            saveBalanceData(newBalance); 
            
            // 3. UI 업데이트
            currentRobuxBalance = newBalance;
            robuxBalanceInput.value = newBalance;
            updateKrwBalance();

            // 4. 성공 처리
            closeTransferModal();
            const successMessage = `${formatNumber(amount)} RBX를 ${recipient}님께 송금 완료! `;
            showSuccessAnimation(successMessage);
            showMessage(successMessage, false);
        };
        // ----------------------


        // =========================================================================
        // 6. 이벤트 리스너 설정
        // =========================================================================

        const setupEventListeners = () => {
            // PIN 입력 및 인증
            authButton.addEventListener('click', handleAuthAction);
            pinInputContainer.addEventListener('keyup', (e) => {
                if (e.key === 'Enter' && !authButton.disabled) {
                    handleAuthAction();
                }
            });

            // 잔액 저장
            robuxBalanceInput.addEventListener('blur', handleBalanceInputSave);
            
            // 지출 저장
            saveButton.addEventListener('click', saveSpendingData);

            // 사이드바
            menuToggleButton.addEventListener('click', toggleSidebar);
            sidebarLogoutButton.addEventListener('click', handleLogout);

            // 송금 모달 열기 버튼
            openTransferModalButtonMain.addEventListener('click', openTransferModal);
            openTransferModalButtonSidebar.addEventListener('click', openTransferModal);

            // 송금 모달 내부 버튼
            cancelTransferButton.addEventListener('click', closeTransferModal);
            confirmTransferButton.addEventListener('click', handleTransfer); // 가상 송금 실행

            // 모달 외부 클릭 시 닫기
            transferModal.addEventListener('click', (e) => {
                if (e.target === transferModal) {
                    closeTransferModal();
                }
            });
            successModal.addEventListener('click', (e) => {
                if (e.target === successModal) {
                    toggleModal(successModal, false);
                }
            });

            // 일반 버튼에 클릭 사운드 일괄 적용 (중복 방지)
             [menuToggleButton, sidebarLogoutButton, saveButton, openTransferModalButtonMain, openTransferModalButtonSidebar].forEach(btn => {
                btn.addEventListener('click', playClickSound); 
            });

            updateKrwBalance();
        };

        window.onload = () => {
            initializeAudio(); // 사운드 초기화
            loadLocalData(); // 로컬 데이터 로드
            setupPinInputs(); // PIN 입력창 생성
            setupEventListeners();
        };

    </script>
</body>
</html>
