# Best Practices
# Star Wars Lego — Greenfield

**Document Version:** 1.0

## 1. JavaScript Development Best Practices

### 1.1 ES6+ Feature Utilization
```javascript
// Preferred modern JavaScript patterns

// Use arrow functions for callbacks
const filterCharacters = (characters, query) => {
  return characters.filter(character =>
    character.name.toLowerCase().includes(query.toLowerCase())
  );
};

// Use destructuring for object properties
const { id, name, image } = character;

// Use template literals for string formation
const characterCardHTML = `
  <div class="character-card" data-id="${id}">
    <img src="${image}" alt="${name}">
    <h3>${name}</h3>
  </div>
`;

// Use spread operator for array operations
const updatedCharacters = [...characters, newCharacter];

// Use const/let appropriately
const BASE_URL = 'https://swapi.dev/api/'; // Immutable
let searchTimeout = null; // Mutable
```

### 1.2 Vue.js Best Practices
```javascript
// Vue.js component best practices

Vue.component('character-gallery', {
  // Explicit prop declarations with types
  props: {
    characters: {
      type: Array,
      required: true,
      validator: value => {
        return value.every(char => 
          char.id && char.name && char.image
        );
      }
    }
  },
  
  // Data must be functions for component instances
  data() {
    return {
      selectedCharacter: null,
      isAnimating: false
    };
  },
  
  // Use computed properties for derived data
  computed: {
    visibleCharacters() {
      return this.characters.filter(char => !char.deleted);
    },
    
    likedCharactersCount() {
      return this.characters.filter(char => char.liked).length;
    }
  },
  
  // Methods for user interactions
  methods: {
    selectCharacter(character) {
      this.selectedCharacter = character;
      this.$emit('character-selected', character);
    },
    
    toggleLike(characterId) {
      const character = this.characters.find(c => c.id === characterId);
      if (character) {
        this.$set(character, 'liked', !character.liked);
        this.showNotification(`${character.name} ${character.liked ? 'liked' : 'unliked'}!`);
      }
    }
  },
  
  // Component template with semantic HTML
  template: `
    <section class="character-gallery" aria-label="Character Collection">
      <div class="gallery-grid">
        <article 
          v-for="character in visibleCharacters"
          :key="character.id"
          class="character-card"
          @click="selectCharacter(character)"
          :aria-label="character.name"
        >
          <img :src="character.image" :alt="character.name" class="character-image">
          <h3 class="character-name">{{ character.name }}</h3>
          <div class="character-actions">
            <button 
              @click.stop="toggleLike(character.id)"
              :aria-pressed="character.liked"
              class="like-button"
            >
              {{ character.liked ? '♥' : '♡' }}
            </button>
          </div>
        </article>
      </div>
    </section>
  `
});
```

### 1.3 Code Organization Best Practices
```javascript
// Separation of concerns in application structure

// 1. Data layer - pure data management
const DataManager = {
  loadCharacters() {
    return [
      { id: 1, name: 'Luke Skywalker', image: 'images/luke.jpg', liked: false, deleted: false },
      { id: 2, name: 'Darth Vader', image: 'images/vader.jpg', liked: false, deleted: false },
      // ... more characters
    ];
  },
  
  saveCharacter(character) {
    localStorage.setItem('character_' + character.id, JSON.stringify(character));
  },
  
  loadCharacter(characterId) {
    const saved = localStorage.getItem('character_' + characterId);
    return saved ? JSON.parse(saved) : null;
  }
};

// 2. Business logic layer - data transformation
const CharacterService = {
  searchCharacters(characters, query) {
    if (!query || query.trim() === '') return characters;
    
    const searchTerm = query.toLowerCase().trim();
    return characters.filter(character => 
      character.name.toLowerCase().includes(searchTerm)
    );
  },
  
  likeCharacter(characters, characterId) {
    const character = characters.find(c => c.id === characterId);
    if (character) {
      character.liked = !character.liked;
      DataManager.saveCharacter(character);
    }
    return characters;
  },
  
  deleteCharacter(characters, characterId) {
    const character = characters.find(c => c.id === characterId);
    if (character) {
      character.deleted = true;
      DataManager.saveCharacter(character);
    }
    return characters;
  }
};

// 3. UI layer - Vue.js application
const app = new Vue({
  el: '#app',
  data: {
    characters: [],
    searchQuery: '',
    isLoading: false
  },
  
  created() {
    this.loadCharacters();
  },
  
  methods: {
    loadCharacters() {
      this.isLoading = true;
      this.characters = DataManager.loadCharacters();
      this.isLoading = false;
    },
    
    onSearchInput() {
      // Debounced search
      clearTimeout(this.searchTimeout);
      this.searchTimeout = setTimeout(() => {
        this.characters = CharacterService.searchCharacters(
          DataManager.loadCharacters(), 
          this.searchQuery
        );
      }, 300);
    },
    
    onCharacterLiked(characterId) {
      this.characters = CharacterService.likeCharacter(this.characters, characterId);
    },
    
    onCharacterDeleted(characterId) {
      if (confirm('Are you sure you want to delete this character?')) {
        this.characters = CharacterService.deleteCharacter(this.characters, characterId);
      }
    }
  }
});
```

## 2. CSS Development Best Practices

### 2.1 CSS Architecture Best Practices
```css
/* CSS custom properties for maintainability */
:root {
  /* Design system tokens */
  --color-bg-primary: #111;
  --color-bg-secondary: #222;
  --color-text-primary: #CCC;
  --color-text-secondary: #999;
  --color-accent-primary: #FD4;
  --color-accent-secondary: #FFD700;
  --color-danger: #FF4444;
  
  /* Typography scale */
  --font-size-base: 1rem;
  --font-size-large: 1.25rem;
  --font-size-small: 0.875rem;
  --font-weight-normal: 400;
  --font-weight-bold: 700;
  
  /* Spacing system */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  /* Layout constraints */
  --container-min-width: 820px;
  --container-max-width: 1200px;
  --border-radius: 0.5rem;
  --transition-fast: 200ms;
  --transition-medium: 300ms;
}

/* Mobile-first responsive design */
.app-container {
  width: 100%;
  max-width: var(--container-max-width);
  margin: 0 auto;
  padding: var(--spacing-md);
  min-width: var(--container-min-width);
}

/* Component-based CSS organization */
.character-gallery {
  display: grid;
  gap: var(--spacing-md);
  padding: var(--spacing-md);
  background: var(--color-bg-secondary);
  border-radius: var(--border-radius);
}

.character-card {
  background: var(--color-bg-primary);
  border: 2px solid transparent;
  border-radius: var(--border-radius);
  padding: var(--spacing-md);
  transition: all var(--transition-medium) ease;
  cursor: pointer;
}

/* State-based styling */
.character-card:hover {
  border-color: var(--color-accent-primary);
  transform: translateY(-4px);
  box-shadow: 0 8px 16px rgba(253, 68, 68, 0.2);
}

.character-card--liked {
  border-color: var(--color-accent-secondary);
}

.character-card--deleted {
  opacity: 0.5;
  pointer-events: none;
}

/* Accessibility-focused CSS */
.character-card:focus-visible {
  outline: 2px solid var(--color-accent-primary);
  outline-offset: 2px;
}

/* Performance-optimized animations */
.character-image {
  transition: transform var(--transition-fast) ease;
  will-change: transform;
}

.character-card:hover .character-image {
  transform: scale(1.05);
}
```

### 2.2 Performance Best Practices
```css
/* Efficient CSS selectors */
/* GOOD: Class-based selectors */
.character-card__title { /* styles */ }
.character-card__image { /* styles */ }

/* AVOID: Deep nesting */
.app .section .gallery .card .title { /* less efficient */ }

/* Minimal specificity */
.button { /* base styles */ }
.button--primary { /* modifiers */ }
.button--large { /* size variants */ }

/* Performance-optimized animations */
.character-card {
  /* Use transform and opacity for smooth animations */
  transition: transform 0.3s ease, opacity 0.3s ease;
  will-change: transform, opacity; /* Hint to browser */
}

/* Avoid animating expensive properties */
.character-card:hover {
  /* GOOD: Transform and opacity */
  transform: translateY(-4px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.2); /* Minimal shadow changes */
  
  /* AVOID: Animating layout properties */
  /* margin-top: -4px; Don't animate this */
  /* width: 110%; Don't animate this */
}
```

## 3. HTML Structure Best Practices

### 3.1 Semantic HTML5 Structure
```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Interactive Star Wars Lego application for learning JavaScript and Vue.js">
  <title>Star Wars Lego - Interactive JavaScript Learning</title>
  
  <!-- Preload critical resources -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Acme&family=Handlee&display=swap" rel="stylesheet">
</head>
<body>
  <div id="app" class="app-container">
    <!-- Application header with accessibility -->
    <header class="app-header" role="banner">
      <h1 class="app-title">Star Wars Lego Gallery</h1>
      <p class="app-subtitle">Interactive JavaScript Learning Experience</p>
    </header>
    
    <!-- Main application content -->
    <main class="app-main" role="main">
      <!-- Search section -->
      <section class="search-section" aria-label="Character Search">
        <label for="character-search" class="search-label">Search Characters:</label>
        <input 
          id="character-search"
          type="text"
          class="search-input"
          placeholder="Enter character name..."
          v-model="searchQuery"
          aria-describedby="search-help"
          required
        >
        <div id="search-help" class="sr-only">Type to filter characters by name</div>
      </section>
      
      <!-- Character gallery -->
      <section class="character-gallery" aria-label="Character Collection">
        <h2 class="section-title">Character Gallery</h2>
        <div class="gallery-grid" role="grid" aria-label="Character cards">
          <!-- Character cards will be dynamically inserted here -->
        </div>
      </section>
      
      <!-- Notification area -->
      <section class="notification-area" aria-live="polite" aria-atomic="true">
        <!-- Notification messages will appear here -->
      </section>
    </main>
    
    <!-- Application footer -->
    <footer class="app-footer" role="contentinfo">
      <p>© 2024 Globant - Educational JavaScript Application</p>
    </footer>
  </div>
</body>
</html>
```

### 3.2 Accessibility Best Practices
```html
<!-- Comprehensive accessibility implementation -->
<div class="character-card" tabindex="0" role="article" aria-label="{{ character.name }}">
  <img 
    :src="character.image" 
    :alt="`Image of ${character.name} from Star Wars Lego`"
    class="character-image"
    loading="lazy"
  >
  
  <div class="character-info">
    <h3 class="character-name">{{ character.name }}</h3>
    
    <div class="character-actions">
      <button 
        @click="toggleLike(character.id)"
        :aria-pressed="character.liked"
        :aria-label="`${character.liked ? 'Remove' : 'Add'} ${character.name} from favorites`"
        class="action-button like-button"
        type="button"
      >
        <span class="sr-only">{{ character.liked ? 'Remove from favorites' : 'Add to favorites' }}</span>
        <span aria-hidden="true">{{ character.liked ? '♥' : '♡' }}</span>
      </button>
      
      <button 
        @click="deleteCharacter(character.id)"
        :aria-label="`Delete ${character.name} from gallery`"
        class="action-button delete-button"
        type="button"
      >
        <span class="sr-only">Delete {{ character.name }}</span>
        <span aria-hidden="true">×</span>
      </button>
    </div>
  </div>
</div>

<!-- Screen reader only content -->
<style>
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Focus management for keyboard navigation */
.character-card:focus-visible {
  outline: 3px solid #FD4;
  outline-offset: 2px;
}

/* High contrast mode support */
@media (prefers-contrast: high) {
  .character-card {
    border: 2px solid #CCC;
  }
}
</style>
```

## 4. Performance Best Practices

### 4.1 Image Optimization
```html
<!-- Optimized image loading strategy -->
<img 
  :src="character.image" 
  :alt="character.name"
  class="character-image"
  loading="lazy"
  decoding="async"
  :style="`aspect-ratio: 1/1; max-height: 200px;`"
>

<!-- Fallback for image loading errors -->
<img 
  :src="character.image" 
  :alt="character.name"
  class="character-image"
  loading="lazy"
  @error="handleImageError"
>
```

### 4.2 Font Loading Optimization
```html
<!-- Optimal font loading strategy -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link 
  href="https://fonts.googleapis.com/css2?family=Acme&family=Handlee&display=swap" 
  rel="stylesheet"
>

<!-- Font display fallback strategy -->
<style>
@font-face {
  font-family: 'Acme';
  src: local('Acme'), local('Acme-Regular');
  font-display: swap; /* Prevents blocking text rendering */
}

/* System font fallbacks */
body {
  font-family: 'Acme', 'Arial', sans-serif;
}

h1, h2, h3 {
  font-family: 'Acme', 'Helvetica', 'Arial', sans-serif;
}
</style>
```

## 5. Security Best Practices

### 5.1 Input Validation and Sanitization
```javascript
// Comprehensive input validation
const InputValidator = {
  // Validate search input
  validateSearchInput(input) {
    if (!input || typeof input !== 'string') {
      return { valid: false, message: 'Input is required' };
    }
    
    const sanitized = input.trim().substring(0, 100);
    
    // Remove potentially dangerous characters
    const cleaned = sanitized.replace(/[<>\"']/g, '');
    
    if (cleaned.length === 0) {
      return { valid: false, message: 'Search query cannot be empty' };
    }
    
    return { valid: true, value: cleaned };
  },
  
  // Validate character ID
  validateCharacterId(id) {
    const numId = parseInt(id, 10);
    
    if (isNaN(numId) || numId < 1 || numId > 8) {
      return { valid: false, message: 'Invalid character ID' };
    }
    
    return { valid: true, value: numId };
  },
  
  // Sanitize user-generated content
  sanitizeHTML(input) {
    const div = document.createElement('div');
    div.textContent = input; // Automatically escapes HTML
    return div.innerHTML;
  }
};

// Vue.js input validation integration
const app = new Vue({
  data: {
    searchQuery: '',
    searchError: ''
  },
  
  watch: {
    searchQuery(newValue) {
      const validation = InputValidator.validateSearchInput(newValue);
      if (validation.valid) {
        this.searchError = '';
        this.performSearch(validation.value);
      } else {
        this.searchError = validation.message;
      }
    }
  }
});
```

### 5.2 Safe DOM Manipulation
```javascript
// Safe DOM practices
const SafeDOM = {
  // Use Vue.js templates instead of direct DOM manipulation
  createCharacterCard(character) {
    // AVOID: Direct HTML string concatenation
    // return `<div>${character.name}</div>;` // Unsafe
    
    // GOOD: Use Vue.js template system
    // This template will be safely rendered by Vue.js
    return `
      <div class="character-card" :data-id="${character.id}">
        <h3>{{ character.name }}</h3>
      </div>
    `;
  },
  
  // Handle user input safely
  displayUserMessage(message) {
    // AVOID: innerHTML with user input
    // notificationElement.innerHTML = message; // Unsafe
    
    // GOOD: textContent or Vue.js interpolation
    notificationElement.textContent = message; // Safe
  }
};
```

## 6. Educational Best Practices

### 6.1 Code Documentation for Learning
```javascript
/**
 * Search for characters in the Star Wars Lego collection.
 * This function demonstrates array filtering and string manipulation
 * which are fundamental JavaScript concepts students need to learn.
 * 
 * @param {Array} characters - Array of character objects with id, name, and image properties
 * @param {string} query - Search text entered by the user
 * @returns {Array} Filtered array of characters matching the search criteria
 * 
 * @example
 * // Example usage in the application:
 * const allCharacters = [
 *   { id: 1, name: 'Luke Skywalker', image: 'luke.jpg' },
 *   { id: 2, name: 'Darth Vader', image: 'vader.jpg' }
 * ];
 * const results = searchCharacters(allCharacters, 'Luke');
 * console.log(results); // Returns only Luke Skywalker
 */
function searchCharacters(characters, query) {
  // Learning concepts demonstrated:
  // 1. Input validation - checking if input is valid
  // 2. String manipulation - toLowerCase() for case-insensitive search
  // 3. Array methods - filter() for creating new arrays
  // 4. Conditional logic - short-circuiting for empty queries
  
  if (!query || query.trim() === '') {
    return characters; // Return all characters if search is empty
  }
  
  // Convert search query to lowercase for case-insensitive comparison
  const lowercasedQuery = query.toLowerCase().trim();
  
  // Use the filter() array method to create a new array containing only
  // characters whose names include the search text
  return characters.filter(character => {
    // The includes() method checks if one string can be found within another
    return character.name.toLowerCase().includes(lowercasedQuery);
  });
}
```

### 6.2 Progressive Learning Structure
```javascript
// Start with basic concepts and progressively introduce complexity

// LEVEL 1: Basic array operations
const basicSearch = (characters, query) => {
  // Simple iteration with for loop
  const results = [];
  const lowerQuery = query.toLowerCase();
  
  for (let i = 0; i < characters.length; i++) {
    if (characters[i].name.toLowerCase().includes(lowerQuery)) {
      results.push(characters[i]);
    }
  }
  
  return results;
};

// LEVEL 2: Introduce modern array methods
const intermediateSearch = (characters, query) => {
  // Use filter() method - more concise
  return characters.filter(character => 
    character.name.toLowerCase().includes(query.toLowerCase())
  );
};

// LEVEL 3: Add performance optimization
const advancedSearch = (characters, query, searchOptions = {}) => {
  const {
    caseSensitive = false,
    exactMatch = false,
    debounceMs = 300
  } = searchOptions;
  
  // Debounce for performance
  if (this.searchTimeout) {
    clearTimeout(this.searchTimeout);
  }
  
  this.searchTimeout = setTimeout(() => {
    let filteredCharacters = characters;
    
    if (!caseSensitive) {
      query = query.toLowerCase();
      filteredCharacters = filteredCharacters.map(char => ({
        ...char,
        name: char.name.toLowerCase()
      }));
    }
    
    if (exactMatch) {
      filteredCharacters = filteredCharacters.filter(char => 
        char.name === query
      );
    } else {
      filteredCharacters = filteredCharacters.filter(char => 
        char.name.includes(query)
      );
    }
    
    return filteredCharacters;
  }, debounceMs);
};
```

---

*Star Wars Lego Best Practices — Globant Delivery Team*