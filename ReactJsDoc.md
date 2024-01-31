## ReactJs Doc

#### React App

```bash
npm create vite
```
give an app name, and select the others option. Go to project folder, then

```bash
npm install
npm run dev
```

#### React Router

```bash
npm install react-router-dom
```

#### React Icons

```bash
npm install react-icons --save
```

#### Tailwind CSS

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```bash
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

```bash
@tailwind base;
@tailwind components;
@tailwind utilities;
```
