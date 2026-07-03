# Coding Standards
# Star Wars Lego — Greenfield

**Document Version:** 1.0

## 1. JavaScript ES6+ Coding Standards

### 1.1 Variable Declaration Standards
```javascript
// Preferred: const for immutable variables
const MAX_CHARACTERS = 8;
const API_BASE_URL = 'https://api.star-wars.com';
const SEARCH_DEBOUNCE_MS = 300;

// Use let for variables that need to be reassigned
let searchTimeout = null;
let selectedCharacterId = null;
let isDataLoading = false;

// Avoid var - use modern block-scoped declarations
// BAD: var characterId = 1;  // Function-scoped, can cause confusion

// Naming conventions
const UPPER_SNAKE_CASE = 'constants'; // Constants
const camelCaseVariables = 'variables and functions'; // Variables and functions
const PascalCaseClass = 'ClassNames'; // Classes/Constructors

// Descriptive variable names
const userSearchQuery = ''; // GOOD: descriptive
const s = ''; // BAD: cryptic
const data = ''; // BAD: too generic
```

### 1.2 Function Declaration Standards
```javascript
// Use arrow functions for callbacks and short functions
const searchCharacters = (characters, query) => {
  return characters.filter(character => 
    character.name.toLowerCase().includes(query.toLowerCase())
  );
};

// Use function declarations for named functions
const CharacterManager = {
  loadCharacters() {
    // Implementation
  },
  
  saveCharacter(character) {
    // Implementation
  }
};

// Method naming - use verbs for actions
const handleCharacterClick = (characterId) => {
  selectCharacter(characterId);
};

const toggleCharacterLike = (characterId) => {
  const character = findCharacterById(characterId);
  character.liked = !character.liked;
  updateCharacterInStorage(character);
};

// Avoid anonymous functions for better debugging
// BAD: setTimeout(() => { /* complex logic */ }, 1000);
// GOOD: setTimeout(handleSearchDebounce, 1000);
```

### 1.3 Array and Object Manipulation Standards
```javascript
// Modern array methods preferred over traditional loops
const characters = ['Luke', 'Leia', 'Han'];

// GOOD: map for transformation
const characterObjects = characters.map(name => ({
  id: generateId(),
  name: name,
  liked: false,
  deleted: false
}));

// GOOD: filter for selection
const activeCharacters = characterObjects.filter(char => !char.deleted);

// GOOD: find for single item lookup
const luke = characterObjects.find(char => char.name.includes('Luke'));

// GOOD: some/any for existence check
const hasLikedCharacters = characterObjects.some(char => char.liked);

// Object destructuring for clean code
const { id, name, image } = character;
const { characters, searchQuery, isLoading } = this;

// Spread operator for immutable updates
const updatedCharacters = [...characters, newCharacter];
const deletedCharacter = {
  ...character,
  deleted: true,
  deletedAt: new Date().toISOString()
};

// Good: Destructuring with default values
const { color = 'default', size = 'medium' } = config;

// Avoid mutating function parameters directly
// BAD: function updateCharacter(character) { character.liked = true; }
// GOOD: function updateCharacter(character) { return { ...character, liked: true }; }
```

### 1.4 Error Handling Standards
```javascript
// Try-catch blocks for error-prone operations
const loadCharacterData = async () => {
  try {
    const response = await fetch('/api/characters');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data.characters;
  } catch (error) {
    console.error('Failed to load character data:', error);
    showNotification('Unable to load characters. Please try again.', 'error');
    return getFallbackCharacterData(); // Provide fallback
  }
};

// Validation functions with clear error messages
const validateCharacterId = (id) => {
  if (!id) {
    throw new Error('Character ID is required');
  }
  
  const numId = parseInt(id, 10);
  if (isNaN(numId)) {
    throw new Error('Character ID must be a number');
  }
  
  if (numId < 1 || numId > 8) {
    throw new Error('Character ID must be between 1 and 8');
  }
  
  return numId;
};

// Safe property access with optional chaining
const userName = user?.profile?.name ?? 'Anonymous';
const characterImage = character?.images?.large ?? character?.image ?? 'default.jpg';
```

## 2. Vue.js Specific Coding Standards

### 2.1 Component Definition Standards
```javascript
// Vue.js component structure template
Vue.component('character-card', {
  // 1. Props with validation
  props: {
    character: {
      type: Object,
      required: true,
      validator: character => {
        const requiredFields = ['id', 'name', 'image'];
        return requiredFields.every(field => character[field]);
      }
    },
    
    // Optional props with defaults
    size: {
      type: String,
      default: 'medium',
      validator: size => ['small', 'medium', 'large'].includes(size)
    },
    
    // Complex prop with custom validator
    config: {
      type: Object,
      default: () => ({
        showActions: true,
        animateOnHover: true,
        lazyLoadImages: true
      })
    }
  },
  
  // 2. Data function for component state
  data() {
    return {
      isHovered: false,
      isAnimating: false,
      imageError: false
    };
  },
  
  // 3. Computed properties for derived data
  computed: {
    characterDisplayName() {
      return this.character.name.trim();
    },
    
    isLiked() {
      return this.character.liked === true;
    },
    
    cardClasses() {
      return {
        'character-card': true,
        'character-card--hovered': this.isHovered,
        'character-card--liked': this.isLiked,
        'character-card--animating': this.isAnimating,
        [`character-card--${this.size}`]: this.size !== 'medium'
      };
    }
  },
  
  // 4. Methods for user interactions
  methods: {
    // Event handlers should be prefixed with 'handle'
    handleCardClick() {
      this.$emit('click', this.character);
    },
    
    handleLikeClick(event) {
      // Prevent event bubbling
      event.stopPropagation();
      
      if (this.isAnimating) return; // Prevent rapid clicks
      
      this.startAnimation();
      this.$emit('like', this.character.id);
    },
    
    // Internal methods should be descriptive
    startAnimation() {
      this.isAnimating = true;
      setTimeout(() => {
        this.isAnimating = false;
      }, 300);
    },
    
    // Service calls should be clearly named
    async loadCharacterDetails() {
      try {
        const details = await CharacterApi.getDetails(this.character.id);
        this.characterDetails = details;
      } catch (error) {
        console.error('Failed to load character details:', error);
      }
    }
  },
  
  // 5. Component lifecycle hooks
  created() {
    this.initializeComponent();
  },
  
  mounted() {
    this.setupEventListeners();
  },
  
  beforeDestroy() {
    this.cleanupEventListeners();
  },
  
  // 6. Watchers for reactive data
  watch: {
    'character.id': {
      immediate: true,
      handler(newId) {
        if (newId) {
          this.loadCharacterDetails();
        }
      }
    },
    
    isHovered(newValue) {
      if (newValue && this.config.animateOnHover) {
        this.startHoverAnimation();
      }
    }
  },
  
  // 7. Component template (can be extracted to separate method)
  template: `
    <article 
      :class="cardClasses"
      @click="handleCardClick"
      @mouseenter="isHovered = true"
      @mouseleave="isHovered = false"
    >
      <img 
        :src="character.image" 
        :alt="character.name"
        @error="handleImageError"
        v-if="!imageError"
      >
      <div v-else class="image-error">Image unavailable</div>
      
      <h3>{{ characterDisplayName }}</h3>
      
      <button 
        v-if="config.showActions"
        @click="handleLikeClick"
        :class="{ 'liked': isLiked }"
      >
        {{ isLiked ? '♥' : '♡' }}
      </button>
    </article>
  `
});
```

### 2.2 Vue.js Template Standards
```html
<!-- Use semantic HTML5 elements in templates -->
<section class="character-gallery" aria-label="Character Collection">
  <header class="gallery-header">
    <h2>Character Gallery</h2>
    <p class="gallery-subtitle">Interactive Star Wars Lego Collection</p>
  </header>
  
  <!-- Use v-for with explicit :key -->
  <div class="gallery-grid">
    <character-card
      v-for="character in visibleCharacters"
      :key="character.id"
      :character="character"
      @click="selectCharacter"
      @like="toggleLike"
    />
  </div>
  
  <!-- Use conditional rendering appropriately -->
  <div v-if="isLoading" class="loading-state">
    <span aria-hidden="true">Loading characters...</span>
  </div>
  
  <div v-else-if="visibleCharacters.length === 0" class="empty-state">
    <h3>No characters found</h3>
    <p>Try adjusting your search criteria.</p>
  </div>
  
  <!-- Use v-show for frequent toggling -->
  <footer v-show="showFooter" class="gallery-footer">
    <p>{{ visibleCharacters.length }} characters displayed</p>
  </footer>
</section>
```

## 3. CSS Coding Standards

### 3.1 CSS Architecture and Organization
```css
/* Use CSS custom properties for maintainability */
:root {
  /* Color palette - semantic naming */
  --color-bg-primary: #111;           /* Main background */
  --color-bg-secondary: #222;         /* Card/section backgrounds */
  --color-text-primary: #CCC;         /* Main text */
  --color-text-secondary: #999;       /* Secondary text */
  --color-accent-primary: #FD4;       /* Star Wars yellow */
  --color-accent-secondary: #FFD700;   /* Alternative accent */
  --color-danger: #FF4444;            /* Error/deletion states */
  --color-success: #4CAF50;           /* Success states */
  --color-border: #444;               /* Border color */
  
  /* Typography scale */
  --font-family-primary: 'Acme', sans-serif;
  --font-family-secondary: 'Handlee', cursive;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  
  /* Spacing system */
  --spacing-xs: 0.25rem;  /* 4px */
  --spacing-sm: 0.5rem;   /* 8px */
  --spacing-md: 1rem;     /* 16px */
  --spacing-lg: 1.5rem;   /* 24px */
  --spacing-xl: 2rem;     /* 32px */
  --spacing-2xl: 3rem;    /* 48px */
  
  /* Layout constraints */
  --container-max-width: 1200px;
  --container-min-width: 820px;
  --border-radius-base: 0.5rem;
  --border-radius-large: 1rem;
  
  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-base: 300ms ease;
  --transition-slow: 500ms ease;
}

/* Component-based CSS structure */
.character-card {
  /* Base component styles */
  background: var(--color-bg-primary);
  border: 2px solid transparent;
  border-radius: var(--border-radius-base);
  padding: var(--spacing-md);
  transition: all var(--transition-base) ease;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  gap: var(--spacing-sm);
}

/* Component elements */
.character-card__image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-radius: var(--border-radius-base);
  transition: transform var(--transition-fast) ease;
}

.character-card__title {
  font-family: var(--font-family-primary);
  font-size: var(--font-size-lg);
  color: var(--color-text-primary);
  margin: 0;
  text-align: center;
}

.character-card__actions {
  display: flex;
  gap: var(--spacing-sm);
  justify-content: center;
}

/* Component modifiers (BEM methodology) */
.character-card--hovered {
  border-color: var(--color-accent-primary);
  transform: translateY(-4px);
  box-shadow: 0 8px 16px rgba(253, 68, 68, 0.2);
}

.character-card--liked {
  border-color: var(--color-accent-secondary);
}

.character-card--large .character-card__image {
  height: 250px;
}

.character-card--small {
  padding: var(--spacing-sm);
}

.component-card__action-button {
  background: transparent;
  border: 1px solid var(--color-border);
  color: var(--color-text-primary);
  padding: var(--spacing-xs) var(--spacing-sm);
  border-radius: var(--border-radius-base);
  cursor: pointer;
  transition: all var(--transition-fast) ease;
}

.character-card__action-button:hover {
  background: var(--color-accent-primary);
  border-color: var(--color-accent-primary);
  color: var(--color-bg-primary);
}
```

### 3.2 Responsive Design Standards
```css
/* Mobile-first responsive design approach */
.app-container {
  width: 100%;
  max-width: var(--container-max-width);
  margin: 0 auto;
  padding: var(--spacing-md);
  min-width: var(--container-min-width);
}

/* Base styles (mobile-first) */
.character-gallery {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--spacing-md);
}

/* Tablet breakpoint */
@media (min-width: 768px) {
  .character-gallery {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop breakpoint (minimum required) */
@media (min-width: 820px) {
  .character-gallery {
    grid-template-columns: repeat(3, 1fr);
  }
  
  .app-container {
    padding: var(--spacing-lg);
  }
  
  .character-card__image {
    height: 220px;
  }
}

/* Large desktop */
@media (min-width: 1200px) {
  .character-gallery {
    grid-template-columns: repeat(4, 1fr);
  }
  
  .character-card__image {
    height: 200px;
  }
}

/* High-DPI displays */
@media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
  .character-card__image {
    image-rendering: -webkit-optimize-contrast;
  }
}

/* Reduced motion preferences */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

## 4. File Naming and Organization Standards

### 4.1 Directory Structure Standards
```
js/
├── app.js                    # Application entry point
├── components/               # Vue.js components
│   ├── CharacterGallery.js   # PascalCase for component files
│   ├── SearchBox.js
│   ├── CharacterCard.js
│   └── NotificationSystem.js
├── services/                 # Business logic services
│   ├── CharacterService.js   # PascalCase with Service suffix
│   ├── SearchService.js
│   └── StorageService.js
├── utils/                    # Utility functions
│   ├── helpers.js            # camelCase for utility files
│   ├── validators.js
│   └── constants.js
├── data/                     # Data definitions
│   ├── characters.js         # Plural for data collections
│   └── config.js
└── mixins/                   # Vue.js mixins (if needed)
    └── AnimationMixin.js
```

```css
css/
├── main.css                  # Entry point
├── utilities.css             # CSS custom properties, utilities
├── components.css            # Component styles
├── layout.css                # Grid, layout systems
└── responsive.css            # Media queries
```

### 4.2 File Content Organization
```javascript
// js/components/CharacterCard.js
/**
 * Character Card Component
 * 
 * @description Individual character display card with like/delete actions
 * @author Globant Delivery Team
 * @version 1.0.0
 */

import { INPUT_VALIDATOR } from '../utils/validators.js';
import { ANIMATION_HELPER } from '../utils/helpers.js';

// Component configuration constants
const COMPONENT_CONFIG = {
  ANIMATION_DURATION: 300,
  MAX_CHARACTER_NAME_LENGTH: 50,
  DEFAULT_IMAGE_PATH: '/assets/images/default-character.jpg'
};

// Component export (for module systems)
export default {
  name: 'CharacterCard',
  // ... component definition
};

// Standalone component registration (for non-module environments)
if (typeof Vue !== 'undefined') {
  Vue.component('character-card', componentDefinition);
}
```

## 5. Documentation and Comments Standards

### 5.1 JSDoc Documentation Standards
```javascript
/**
 * Searches for characters based on a query string
 * 
 * Performs case-insensitive search across character names using
 * modern JavaScript array filtering methods.
 * 
 * @param {Array<Object>} characters - Array of character objects to search
 * @param {string} query - Search query string for filtering characters
 * @param {Object} [options={}] - Optional search configuration
 * @param {boolean} [options.caseSensitive=false] - Whether to perform case-sensitive search
 * @param {boolean} [options.exactMatch=false] - Whether to require exact name match
 * @returns {Array<Object>} Filtered array of characters matching the search criteria
 * 
 * @throws {TypeError} If characters is not an array or query is not a string
 * 
 * @example
 * // Basic usage
 * const results = searchCharacters(characters, 'luke');
 * console.log(results); // [{ id: 1, name: 'Luke Skywalker', ... }]
 * 
 * @example
 * // Advanced usage with options
 * const results = searchCharacters(characters, 'Luke', { exactMatch: true });
 */
export const searchCharacters = (characters, query, options = {}) => {
  // Validate input parameters
  if (!Array.isArray(characters)) {
    throw new TypeError('characters must be an array');
  }
  
  if (typeof query !== 'string') {
    throw new TypeError('query must be a string');
  }
  
  const { caseSensitive = false, exactMatch = false } = options;
  
  // Process query
  const processedQuery = caseSensitive ? query : query.toLowerCase().trim();
  
  if (!processedQuery) {
    return characters; // Return all characters if query is empty
  }
  
  // Filter characters based on options
  return characters.filter(character => {
    const characterName = caseSensitive 
      ? character.name 
      : character.name.toLowerCase();
    
    return exactMatch 
      ? characterName === processedQuery
      : characterName.includes(processedQuery);
  });
};
```

### 5.2 Code Comment Standards
```javascript
// File-level comment explaining purpose
/**
 * Character Service
 * 
 * Handles all business logic related to character management:
 * - Data loading and saving
 * - Character interactions (like, delete)
 * - State synchronization with storage
 */

// Section comments to organize code
// =================================
// Loading and Data Management
// =================================

const loadCharacters = async () => {
  try {
    // Load base character data from local file
    // Note: This would be replaced with API call in production
    const baseCharacters = await import('../data/characters.js');
    
    // Merge with user interactions from localStorage
    const userInteractions = loadUserInteractions();
    return mergeCharactersWithInteractions(baseCharacters.default, userInteractions);
  } catch (error) {
    console.error('Failed to load character data:', error);
    // Fall back to hardcoded data to ensure app works
    return getFallbackCharacterData();
  }
};

// Inline comments for complex logic
const toggleCharacterLike = (characterId) => {
  const character = characters.find(c => c.id === characterId);
  if (!character) {
    // Character not found - this should never happen but we handle it gracefully
    console.warn(`Attempted to like non-existent character: ${characterId}`);
    return;
  }
  
  // Toggle the liked status and update UI immediately
  character.liked = !character.liked;
  
  // Persist the change to localStorage for data persistence
  saveCharacterData(character.id, { liked: character.liked });
  
  // Show user feedback
  showNotification(
    `${character.name} ${character.liked ? 'liked!' : 'unliked!'}`,
    character.liked ? 'success' : 'info'
  );
};

// TODO comments for future work
// TODO: Implement undo functionality for character deletion
// FIXME: Handle edge case where character image fails to load
// NOTE: This function should be moved to a dedicated SearchService
```

## 6. Performance and Optimization Standards

### 6.1 JavaScript Performance Standards
```javascript
// Debouncing for performance optimization
const createDebouncedFunction = (func, delay) => {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
};

// Use in Vue.js components for search
methods: {
  // Debounced search to prevent excessive filtering
  onSearchInput: createDebounced(function(query) {
    this.performSearch(query);
  }, 300)
}

// Efficient DOM manipulation
const updateCharacterGallery = (characters) => {
  // BAD: Direct DOM manipulation in loop
  // characters.forEach(char => {
  //   const element = document.getElementById(`char-${char.id}`);
  //   element.innerHTML = createCharacterHTML(char);
  // });
  
  // GOOD: Let Vue.js handle DOM updates efficiently
  this.characters = characters;
  // Vue.js will batch DOM updates and use virtual DOM diffing
};

// Memory management
const ComponentManager = {
  eventListeners: new Map(),
  
  addEventListener(element, event, handler) {
    const wrappedHandler = handler.bind(this);
    element.addEventListener(event, wrappedHandler);
    
    // Store reference for cleanup
    this.eventListeners.set(element, { event, handler: wrappedHandler });
  },
  
  removeAllEventListeners() {
    this.eventListeners.forEach((value, element) => {
      element.removeEventListener(value.event, value.handler);
    });
    this.eventListeners.clear();
  }
};

// Component lifecycle cleanup
beforeDestroy() {
  // Cancel any pending operations
  if (this.searchTimeout) {
    clearTimeout(this.searchTimeout);
  }
  
  // Remove event listeners
  this.componentManager.removeAllEventListeners();
  
  // Clean up any subscriptions or intervals
  if (this.dataSubscription) {
    this.dataSubscription.unsubscribe();
  }
}
```

### 6.2 CSS Performance Standards
```css
/* Use efficient selectors */
/* GOOD: Class-based selectors */
.character-card { /* styles */ }
.character-card__title { /* styles */ }

/* AVOID: Overly specific selectors */
.app .section .gallery .card .title { /* less efficient */ }

/* Performance-optimized animations */
.character-image {
  /* Use transform and opacity for smooth 60fps animations */
  transition: transform 0.3s ease, opacity 0.3s ease;
  will-change: transform; /* Hint to browser for optimization */
}

.character-card:hover .character-image {
  /* GOOD: Transform-based animations */
  transform: scale(1.05);
  
  /* AVOID: Animating expensive layout properties */
  /* width: 110%; - Causes reflow */
  /* margin-top: -5px; - Causes layout recalculations */
}

/* Efficient CSS for animations */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.character-enter-active {
  animation: fadeIn 0.3s ease;
}

/* Use contain property for performance optimization */
.character-gallery {
  /* Optimize layout painting */
  contain: content layout;
}
```

---

*Star Wars Lego Coding Standards — Globant Delivery Team*