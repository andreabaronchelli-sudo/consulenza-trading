# consulenza-trading
Consulenza Trading con Andrea Baronchelli
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chat Trading Online</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8;
            color: #333;
        }
        .chat-container {
            height: 50vh;
            overflow-y: auto;
            border: 1px solid #e2e8f0;
            border-radius: 0.75rem;
            padding: 1.5rem;
            background-color: white;
            display: flex;
            flex-direction: column;
            gap: 1rem;
        }
        .message {
            max-width: 75%;
            padding: 0.75rem 1rem;
            border-radius: 1rem;
        }
        .user-message {
            align-self: flex-end;
            background-color: #3b82f6;
            color: white;
            border-bottom-right-radius: 0.25rem;
        }
        .ai-message {
            align-self: flex-start;
            background-color: #e5e7eb;
            color: #4b5563;
            border-bottom-left-radius: 0.25rem;
        }
        .loading-dot {
            width: 8px;
            height: 8px;
            background-color: #6b7280;
            border-radius: 50%;
            animation: bounce 1.4s infinite ease-in-out both;
        }
        .loading-dot:nth-child(1) { animation-delay: -0.32s; }
        .loading-dot:nth-child(2) { animation-delay: -0.16s; }
        @keyframes bounce {
            0%, 80%, 100% { transform: translateY(-0); }
            40% { transform: translateY(-8px); }
        }
    </style>
</head>
<body class="flex flex-col items-center justify-center min-h-screen p-4 sm:p-6">

    <div class="w-full max-w-2xl bg-white p-8 rounded-xl shadow-lg border border-gray-200">
        <h1 class="text-3xl font-bold text-center text-gray-800 mb-2">Consulenza Trading con Andrea Baronchelli</h1>
        <p class="text-center text-gray-500 mb-6">Broker per **CC INVEST** - Sede a Londra. Chiedi al nostro AI.</p>

        <div id="chat-container" class="chat-container mb-6">
            <div class="ai-message message shadow-sm">
                Benvenuto! Sono il tuo assistente per il trading online. Fammi una domanda, ad esempio: "Cos'è il trading online?" o "Quali sono i rischi?".
            </div>
        </div>

        <form id="chat-form" class="flex items-center gap-4">
            <input type="text" id="chat-input" placeholder="Scrivi qui la tua domanda..."
                   class="flex-1 p-4 border border-gray-300 rounded-full focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors">
            <button type="submit" id="send-btn"
                    class="p-4 bg-blue-600 text-white rounded-full shadow-lg hover:bg-blue-700 transition-colors focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-6 h-6">
                    <path d="M3.478 2.405a.75.75 0 00-.926.94l2.432 7.945a.75.75 0 00.926.94L20.895 18.2a.75.75 0 00.926-.94l-2.432-7.945a.75.75 0 00-.926-.94L3.478 2.405z" />
                </svg>
            </button>
        </form>
    </div>

    <script type="module">
        const chatContainer = document.getElementById('chat-container');
        const chatForm = document.getElementById('chat-form');
        const chatInput = document.getElementById('chat-input');
        const sendBtn = document.getElementById('send-btn');

        function addMessageToChat(message, isUser = false) {
            const messageElement = document.createElement('div');
            messageElement.textContent = message;
            messageElement.className = `message shadow-sm ${isUser ? 'user-message' : 'ai-message'}`;
            chatContainer.appendChild(messageElement);
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        function showLoadingIndicator() {
            const loadingElement = document.createElement('div');
            loadingElement.id = 'loading-indicator';
            loadingElement.className = 'flex gap-1.5 p-3 rounded-lg bg-gray-200 w-20 justify-center';
            loadingElement.innerHTML = `
                <div class="loading-dot"></div>
                <div class="loading-dot"></div>
                <div class="loading-dot"></div>
            `;
            chatContainer.appendChild(loadingElement);
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        function hideLoadingIndicator() {
            const loadingIndicator = document.getElementById('loading-indicator');
            if (loadingIndicator) {
                loadingIndicator.remove();
            }
        }

        async function fetchAIResponse(prompt) {
            try {
                const systemPrompt = "Sei un assistente virtuale per il trading online, parte del team di Andrea Baronchelli di CC INVEST con sede a Londra. Rispondi alle domande in modo chiaro, conciso e in italiano. Spiega i concetti in modo semplice. Non dare consigli finanziari, ma solo informazioni generali. Se ti vengono chieste previsioni o raccomandazioni, rispondi che non puoi fornirle. Se non conosci la risposta, invita l'utente a consultare un consulente finanziario professionista.";
                const userQuery = prompt;
                const apiKey = ""
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

                const payload = {
                    contents: [{ parts: [{ text: userQuery }] }],
                    tools: [{ "google_search": {} }],
                    systemInstruction: {
                        parts: [{ text: systemPrompt }]
                    },
                };

                let response;
                let retryCount = 0;
                const maxRetries = 3;
                while (retryCount < maxRetries) {
                    try {
                        response = await fetch(apiUrl, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });
                        if (response.status === 429) {
                            const delay = Math.pow(2, retryCount) * 1000;
                            retryCount++;
                            await new Promise(res => setTimeout(res, delay));
                        } else {
                            break;
                        }
                    } catch (err) {
                        const delay = Math.pow(2, retryCount) * 1000;
                        retryCount++;
                        await new Promise(res => setTimeout(res, delay));
                    }
                }

                if (!response || !response.ok) {
                    throw new Error(`API call failed with status: ${response ? response.status : 'N/A'}`);
                }

                const result = await response.json();
                const candidate = result.candidates?.[0];

                if (candidate && candidate.content?.parts?.[0]?.text) {
                    return candidate.content.parts[0].text;
                } else {
                    console.error("Risposta non valida dall'API:", JSON.stringify(result, null, 2));
                    return "Mi dispiace, non sono riuscito a generare una risposta. Riprova più tardi.";
                }
            } catch (error) {
                console.error("Errore nel recupero della risposta AI:", error);
                return "Si è verificato un errore. Per favore, riprova più tardi.";
            }
        }

        chatForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const prompt = chatInput.value.trim();
            if (prompt) {
                addMessageToChat(prompt, true);
                chatInput.value = '';
                showLoadingIndicator();
                const aiResponse = await fetchAIResponse(prompt);
                hideLoadingIndicator();
                addMessageToChat(aiResponse);
            }
        });
    </script>
</body>
</html>
