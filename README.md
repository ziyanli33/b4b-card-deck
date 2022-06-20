# Back 4 Blood Card Deck
This is a fully localized [Back 4 Blood](https://back4blood.com
) Card Deck Drawing simulator. You can share the card combo link to anyone.

## Get Started
### Development Setup
1. Run the following command at the project root directory
```
npm run dev
```

2. Visit `http://localhost:3000` at your favourite browser(Chrome, Edge, Firefox etc.)
### Container Setup
1. Run the following command at the project root directory
```
docker build -t b4bcarddeck .
docker run -p 80:80 -d b4bcarddeck
```

2. Visit `http://localhost` at your favourite browser(Chrome, Edge, Firefox etc.)

## Contributing
### Vue 3 + TypeScript

The project uses Vue 3 `<script setup>` SFCs and Typescript.

### Recommended IDE Setup

- [VS Code](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar)
