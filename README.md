Dieu-Merci Puzzle Game 🧩
A dynamic, interactive sliding-tile puzzle game engineered using the Vue.js framework. This project demonstrates modular component architecture, reactive state management, event-driven audio integration, and decoupled game matrix verification logic.

🚀 Architectural & Technical Features
Reactive Vue State Logic: Leverages Vue’s reactivity system to track matrix coordinates, tile positions, and game-win states cleanly without direct DOM manipulation.

Automated Audio Pipeline: Features decoupled background ambient tracks and event-driven audio clips (.wav/.mp3) mapped dynamically to user interactions like tile swaps and victory configurations.

Dynamic Level Progression: Structured level-handling logic that efficiently unmounts the current puzzle matrix, handles image asset swapping, and instantiates the next stage configuration seamlessly.

💻 Tech Stack
Framework: Vue.js (Vue CLI)

Language: JavaScript (ES6+)

Styling & Layout: HTML5 / CSS Grid & Flexbox

Tooling: Babel, ESLint

Version Control: Git / GitHub Flow

🛠️ Project Setup & Local Development
To run this project locally, clone the repository and execute the following commands in your terminal:

1. Install Dependencies
Installs Vue core packages and project configuration dependencies:

Bash
npm install
2. Compiles and Hot-Reloads for Development
Launches a local development server with live-reloading features for rapid iteration:

Bash
npm run serve
3. Compiles and Minifies for Production
Bundles and optimizes production-ready assets into the /dist directory for clean deployment:

Bash
npm run build
4. Lints and Fixes Files
Runs automated static code analysis to enforce clean coding standards and uniform formatting:

Bash
npm run lint
### ⚙️ Customize Configuration
See the official [Configuration Reference](https://cli.vuejs.org/config/).
