# Voyage - Your Journey Visualized

A stunning web application that transforms your travel stories into interactive visualizations on a spinning 3D Earth globe.

## Features

- **Spinning 3D Earth Globe** - Beautiful, interactive globe built with Three.js
- **Journey Story Input** - Write your travel experiences in natural language
- **AI-Powered Processing** - Support for Gemini, Claude, and OpenAI APIs
- **API Key Validation** - Automatic validation and fallback selection
- **Interactive Globe Visualization** - See visited countries, cities, and travel routes
- **Playback Controls** - Play, pause, step-by-step, and rewind your journey
- **Location Cards** - Detailed information about each location visited
- **Responsive Design** - Beautiful dark modern theme that works everywhere

## Tech Stack

- **Frontend**: Vue 3
- **Build Tool**: Vite
- **3D Graphics**: Three.js
- **State Management**: Pinia
- **HTTP Client**: Axios

## Getting Started

### Installation

```bash
npm install
```

### Development

```bash
npm run dev
```

Visit `http://localhost:5173` in your browser.

### Build

```bash
npm run build
```

The built files will be in the `dist/` directory, ready for deployment.

## Usage

1. **Enter Your Journey** - Write a detailed story about your travels
2. **Add API Keys** - Provide at least one valid API key (Gemini, Claude, or OpenAI)
3. **Submit** - Click "Visualize My Journey"
4. **Watch** - See your story come to life on the globe with playback controls

## API Keys

Get your free API keys from:
- **Gemini**: https://makersuite.google.com/app/apikey
- **Claude**: https://console.anthropic.com/
- **OpenAI**: https://platform.openai.com/api-keys

## Project Structure

```
voyage/
├── src/
│   ├── components/        # Vue components
│   ├── services/          # Business logic (AI, geo, rendering)
│   ├── stores/            # Pinia state management
│   ├── styles/            # CSS and animations
│   ├── App.vue
│   └── main.js
├── public/               # Static assets
├── vite.config.js
├── package.json
└── README.md
```

## Future Enhancements

- [ ] Image generation for locations
- [ ] Real-time weather information
- [ ] Journey sharing and collaboration
- [ ] Multiple journey comparison
- [ ] Advanced map layers (elevation, population density)
- [ ] Voice input for stories
- [ ] Mobile app version

## License

MIT