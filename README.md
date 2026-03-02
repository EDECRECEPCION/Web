# Panel de Control de Aulas 🏫⚙️

Una aplicación web Single-Page (SPA) desarrollada en el frontend para gestionar y simular el control de recursos en instalaciones educativas. 

## 🚀 Características Principales

* **Sistema de Autenticación Local:** Flujo de registro e inicio de sesión para alumnos (validado por matrícula). Utiliza `localStorage` y `sessionStorage` para mantener la sesión activa sin necesidad de un backend.
* **Interfaz Dinámica (Grid):** Generación automática de tarjetas de control para 25 áreas distintas (21 Salones convencionales, SUM, Salas de Cómputo y Sala de Maestros).
* **Control de Hardware Simulado:** Botones de acción integrados en cada salón para gestionar el Clima (encendido, apagado, cambio de temperatura) y el equipo multimedia (TV, asistencia HDMI).
* **Buzón de Sugerencias Inteligente:**
  * Límite automatizado de 3 envíos por día, por usuario (calculado mediante la fecha del sistema).
  * Filtro de moderación integrado (array de palabras no permitidas) para evitar lenguaje inapropiado.

## 🛠️ Tecnologías Utilizadas

* **HTML5:** Estructura semántica de la aplicación.
* **CSS3:** Diseño responsivo utilizando variables de entorno (`:root`), Flexbox y CSS Grid.
* **JavaScript (Vanilla):** Lógica de la aplicación, manipulación del DOM, algoritmos de filtrado de arrays (`.some()`, `.includes()`) y gestión de la Web Storage API.

## 💡 Instalación y Uso

Al ser un proyecto estático basado puramente en Frontend, no requiere instalación de dependencias ni configuración de servidores.

1. Clona este repositorio o descarga los archivos.
2. Abre el archivo `index.html` directamente en cualquier navegador web moderno.
3. Para probar el sistema, regístrate con una matrícula inventada (el código de verificación de prueba es `1234`).
