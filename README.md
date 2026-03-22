# 🔓 2-Factor Authentication Component 🔓

![SvelteKit](https://img.shields.io/badge/SvelteKit-1E293B?style=for-the-badge&logo=svelte&logoColor=FF3E00)
![Vite](https://img.shields.io/badge/Vite-2B2D42?style=for-the-badge&logo=vite&logoColor=FFD62E)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-0F172A?style=for-the-badge&logo=tailwindcss&logoColor=38BDF8)
![Vanilla CSS](https://img.shields.io/badge/Vanilla_CSS-1F2937?style=for-the-badge&logo=css3&logoColor=264de4)
![HTML5](https://img.shields.io/badge/HTML5-1E293B?style=for-the-badge&logo=html5&logoColor=E34F26)
![JavaScript ES6+](https://img.shields.io/badge/JavaScript_ES6+-111827?style=for-the-badge&logo=javascript&logoColor=F7DF1E)
![Inline SVGs](https://img.shields.io/badge/Inline_SVGs-0F172A?style=for-the-badge&logo=svg&logoColor=ffffff)
![Google Fonts](https://img.shields.io/badge/Google_Fonts-1F2937?style=for-the-badge&logo=googlefonts&logoColor=4285F4)

- A beautiful, responsive, and accessible 2-Factor Authentication (2FA) component built with SvelteKit, Vite,  
  Tailwind CSS, Vanilla CSS, and JavaScript (ES6+). 
- Enables secure OTP-based authentication for enhanced account protection, with smooth animations, full keyboard 
  navigation, and production-ready security.

🌐 Live Demo : [View Live](https://2factor-authentication.vercel.app/)

## ✨ Features

- 🎨 **Modern UI** - Clean design with smooth animations, responsive layout, and status-based color feedback 
  (default, success, error).
- 🎯 **Micro Interactions** - Animated pulse waves, digit drop effects, and custom cursor animations.
- 🚀 **User Experience** - Keyboard navigation (arrow keys, backspace, tab), paste support, auto-focus, auto-submit,
  and smooth loading states.
- ⚠️ **Error Handling** - Clear visual feedback for invalid OTP with shake animations.
- ♿ **Accessibility** - Screen reader support, high contrast compatibility, full keyboard navigation, and a semantic
  HTML structure for improved accessibility and usability.
- 🔧 **Developer Friendly** - Easy integration, customizable validation, event callbacks, and well-structured
  codebase (TypeScript-ready).

## 🛠️ Tech Stack

- **Framework:** SvelteKit  
- **Build Tool:** Vite  
- **Styling:** Tailwind CSS, Vanilla CSS  
- **Markup:** HTML5  
- **Programming Language:** JavaScript (ES6+)  
- **Assets & Icons:** Inline SVGs  
- **Typography:** Google Fonts  

## 📡 Events

| Event | Payload | Description |
|-------|---------|-------------|
| `success` | `{ code: string }` | Fired on successful verification |
| `error` | `{ code: string, attempts: number }` | Fired on verification failure |
| `input` | `{ code: string, isComplete: boolean }` | Fired on each input change |
| `paste` | `{ code: string }` | Fired when code is pasted |
| `reset` | None | Fired when component is reset |

## 📸 Screenshots

### 🔐 Base State

![Base](./screenshots/base.png)
*Clean and minimal default interface for entering OTP.*

### ✅ Success State

![Success](./screenshots/success.png)
*Smooth visual feedback when authentication is successful.*

### ❌ Error State

![Error](./screenshots/error.png)
*Clear error indication for incorrect OTP input.*

### 📱 Responsive Design

![Responsive](./screenshots/responsive.png)
*Fully optimized layout across mobile, tablet, and desktop devices.*

### 🌞 Light Mode UI

![Light Mode](./screenshots/lightMode.png)
*Elegant and readable light theme experience.* 

## 📦 Setup & Installation

1. **Clone the repository**
```bash
git clone https://github.com/deepanshu1420/2-Factor-Authentication-Component.git
cd 2-Factor-Authentication-Component
```

2. **Install dependencies**
```bash
npm install
# or
yarn install
# or
pnpm install
```

3. **Start development server**
```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

4. **Open in browser**
```
http://localhost:5173
```


