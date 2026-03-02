<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Panel de Control de Aulas</title>
    <style>
        :root {
            --bg-color: #f4f4f9;
            --text-color: #333;
            --blue: #007bff;
            --red: #dc3545;
            --green: #28a745;
        }
        body {
            font-family: Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        h1, h2, h3 { text-align: center; }
        
        /* Formularios de Autenticación */
        .auth-form {
            max-width: 400px;
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        .auth-form input {
            padding: 10px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        .auth-form button {
            padding: 10px;
            font-size: 16px;
            cursor: pointer;
            background-color: #333;
            color: white;
            border: none;
            border-radius: 4px;
        }
        .toggle-auth {
            text-align: center;
            color: var(--blue);
            cursor: pointer;
            text-decoration: underline;
        }

        /* Ocultar secciones */
        .hidden { display: none !important; }

        /* Cuadrícula de Salones */
        .rooms-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
            gap: 15px;
            margin-top: 20px;
        }
        .room-card {
            background: #fdfdfd;
            border: 1px solid #ddd;
            padding: 15px;
            border-radius: 6px;
            text-align: center;
        }
        
        /* Botones de Control */
        .btn {
            display: block;
            width: 100%;
            padding: 8px;
            margin: 5px 0;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            font-size: 14px;
        }
        .btn-blue { background-color: var(--blue); }
        .btn-red { background-color: var(--red); }
        .btn-green { background-color: var(--green); }

        /* Caja de Sugerencias */
        .suggestions-section {
            margin-top: 40px;
            padding-top: 20px;
            border-top: 2px solid #eee;
        }
        textarea {
            width: 100%;
            height: 100px;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            resize: vertical;
        }
        #sug-msg { font-weight: bold; text-align: center; margin-top: 10px; }
        .error { color: var(--red); }
        .success { color: var(--green); }
    </style>
</head>
<body>

<div class="container">
    <div id="auth-section">
        <h1 id="auth-title">Iniciar Sesión</h1>
        
        <div id="login-form" class="auth-form">
            <input type="text" id="login-matricula" placeholder="Matrícula" required>
            <input type="password" id="login-password" placeholder="Contraseña" required>
            <button onclick="login()">Entrar</button>
            <p class="toggle-auth" onclick="toggleAuth()">¿No tienes cuenta? Regístrate aquí</p>
        </div>

        <div id="register-form" class="auth-form hidden">
            <input type="email" id="reg-email" placeholder="Correo Electrónico" required>
            <input type="text" id="reg-matricula" placeholder="Matrícula" required>
            <input type="password" id="reg-password" placeholder="Contraseña" required>
            <button onclick="register()">Registrarse</button>
            <p class="toggle-auth" onclick="toggleAuth()">¿Ya tienes cuenta? Inicia sesión</p>
        </div>
    </div>

    <div id="app-section" class="hidden">
        <div style="display: flex; justify-content: space-between; align-items: center;">
            <h1>Panel de Control</h1>
            <button onclick="logout()" style="padding: 5px 10px; cursor: pointer;">Cerrar Sesión</button>
        </div>
        <h3 id="welcome-msg"></h3>

        <div class="rooms-grid" id="rooms-container">
            </div>

        <div class="suggestions-section">
            <h2>Sugerencias</h2>
            <p style="text-align: center; font-size: 14px; color: #666;">Tienes un límite de 3 sugerencias por día.</p>
            <textarea id="suggestion-box" placeholder="Escribe tu sugerencia aquí..."></textarea>
            <button class="btn btn-blue" style="max-width: 200px; margin: 0 auto;" onclick="submitSuggestion()">Enviar Sugerencia</button>
            <p id="sug-msg"></p>
        </div>
    </div>
</div>

<script>
    /* --- VARIABLES GLOBALES --- */
    let currentUser = null;
    
    // Lista de palabras no permitidas (moderador)
    const badWords = ["pendejo", "idiota", "estupido", "mierda", "cabron", "puto", "chinga"]; 

    // Lista de Salones
    const roomNames = [];
    for(let i = 1; i <= 21; i++) roomNames.push(`Salón ${i}`);
    roomNames.push("SUM", "Computo 1", "Computo 2", "Sala Maestros");

    /* --- INICIALIZACIÓN DE LA INTERFAZ --- */
    window.onload = () => {
        checkSession();
        generateRooms();
    };

    /* --- AUTENTICACIÓN --- */
    function toggleAuth() {
        document.getElementById('login-form').classList.toggle('hidden');
        document.getElementById('register-form').classList.toggle('hidden');
        document.getElementById('auth-title').innerText = 
            document.getElementById('login-form').classList.contains('hidden') ? 'Registro' : 'Iniciar Sesión';
    }

    function register() {
        const email = document.getElementById('reg-email').value;
        const matricula = document.getElementById('reg-matricula').value;
        const password = document.getElementById('reg-password').value;

        if(!email || !matricula || !password) return alert("Llena todos los campos.");

        // Simulación de envío de código
        const code = prompt("Se ha enviado un código a tu correo. (Para esta prueba, ingresa: 1234)");
        
        if (code === "1234") {
            const users = JSON.parse(localStorage.getItem('users')) || {};
            if(users[matricula]) return alert("Esta matrícula ya está registrada.");
            
            users[matricula] = { email, password };
            localStorage.setItem('users', JSON.stringify(users));
            alert("Registro exitoso. Ahora puedes iniciar sesión.");
            toggleAuth();
        } else {
            alert("Código incorrecto. Registro cancelado.");
        }
    }

    function login() {
        const matricula = document.getElementById('login-matricula').value;
        const password = document.getElementById('login-password').value;
        const users = JSON.parse(localStorage.getItem('users')) || {};

        if(users[matricula] && users[matricula].password === password) {
            sessionStorage.setItem('activeUser', matricula);
            checkSession();
        } else {
            alert("Matrícula o contraseña incorrecta.");
        }
    }

    function logout() {
        sessionStorage.removeItem('activeUser');
        checkSession();
    }

    function checkSession() {
        currentUser = sessionStorage.getItem('activeUser');
        if (currentUser) {
            document.getElementById('auth-section').classList.add('hidden');
            document.getElementById('app-section').classList.remove('hidden');
            document.getElementById('welcome-msg').innerText = `Usuario activo: ${currentUser}`;
        } else {
            document.getElementById('auth-section').classList.remove('hidden');
            document.getElementById('app-section').classList.add('hidden');
        }
    }

    /* --- GENERACIÓN DE SALONES Y BOTONES --- */
    function generateRooms() {
        const container = document.getElementById('rooms-container');
        let html = '';

        roomNames.forEach(room => {
            html += `
                <div class="room-card">
                    <h3>${room}</h3>
                    <button class="btn btn-blue" onclick="action('${room}', 'Clima Encendido')">Clima: Encender</button>
                    <button class="btn btn-red" onclick="action('${room}', 'Clima Apagado')">Clima: Apagar</button>
                    <button class="btn btn-green" onclick="action('${room}', 'Temperatura Cambiada')">Cambiar Temperatura</button>
                    <button class="btn btn-blue" onclick="action('${room}', 'TV On/Off')">TV: Encender/Apagar</button>
                    <button class="btn btn-red" onclick="action('${room}', 'Asistencia HDMI Solicitada')">TV: Asistencia HDMI</button>
                </div>
            `;
        });
        container.innerHTML = html;
    }

    function action(room, actionName) {
        alert(`Acción enviada: [${room}] - ${actionName}`);
    }

    /* --- CAJA DE SUGERENCIAS Y MODERADOR --- */
    function submitSuggestion() {
        const text = document.getElementById('suggestion-box').value;
        const msgEl = document.getElementById('sug-msg');
        
        if (!text.trim()) {
            msgEl.innerText = "Por favor, escribe algo antes de enviar.";
            msgEl.className = "error";
            return;
        }

        // 1. Validar límite de usos por día
        const today = new Date().toISOString().split('T')[0]; // Ej: 2026-03-01
        const limits = JSON.parse(localStorage.getItem('suggestionLimits')) || {};
        
        if (!limits[currentUser]) limits[currentUser] = {};
        if (!limits[currentUser][today]) limits[currentUser][today] = 0;

        if (limits[currentUser][today] >= 3) {
            msgEl.innerText = "Has alcanzado el límite de 3 sugerencias por hoy.";
            msgEl.className = "error";
            return;
        }

        // 2. Moderador de palabras
        const textLower = text.toLowerCase();
        const containsBadWord = badWords.some(word => textLower.includes(word));

        if (containsBadWord) {
            msgEl.innerText = "Tu sugerencia contiene lenguaje inapropiado y no puede ser enviada.";
            msgEl.className = "error";
            return;
        }

        // 3. Éxito: Guardar y actualizar límite
        limits[currentUser][today]++;
        localStorage.setItem('suggestionLimits', JSON.stringify(limits));
        
        document.getElementById('suggestion-box').value = '';
        msgEl.innerText = `Sugerencia enviada con éxito. Te quedan ${3 - limits[currentUser][today]} intentos hoy.`;
        msgEl.className = "success";
    }
</script>

</body>
</html>
