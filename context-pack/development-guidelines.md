# Development Guidelines
# Star Wars Lego — Greenfield

**Document Version:** 1.0

## 1. Coding Standards

### JavaScript ES6+ Conventions
- **Naming:** camelCase for variables and functions
- **Constants:** UPPER_SNAKE_CASE for constants
- **Functions:** Clear, descriptive function names
- **Comments:** Educational comments for learning purposes
- **File Organization:** Logical grouping of related functionality

### Vue.js Best Practices
- **Component Names:** kebab-case for component registration
- **Props:** Explicit prop declaration with types
- **Data:** Return functions for component data
- **Methods:** Verb-based method names
- **Templates:** Clean, readable HTML structure

### CSS Guidelines
- **Class Names:** kebab-case for CSS classes
- **Organization:** Logical grouping of styles
- **Responsiveness:** Mobile-first approach (from 820px)
- **Performance:** Efficient selectors and properties

## 2. File Organization Standards

### Proposed Directory Structure
```
star-wars-lego/
├── index.html                    # Main application entry
├── css/
│   ├── reset.css                 # CSS reset (assumption)
│   ├── variables.css             # CSS custom properties
│   ├── layout.css                # Grid and layout styles
│   ├── components.css            # Component-specific styles
│   └── main.css                  # Main stylesheet entry point
├── js/
│   ├── app.js                    # Vue.js application root
│   ├── router.js                 # Route handling (if needed)
│   ├── components/
│   │   ├── CharacterGallery.js   # Character gallery component
│   │   ├── SearchBox.js          # Search functionality
│   │   ├── CharacterCard.js      # Individual character display
│   │   └── Notification.js       # Alert system
│   ├── data/
│   │   └── characters.js         # Character data source
│   └── utils/
│       ├── helpers.js            # Utility functions
│       └── constants.js          # Application constants
├── assets/
│   ├── images/                   # Character and UI images
│   │   ├── characters/          # Character images
│   │   └── ui/                   # UI elements
│   └── icons/                    # Icon resources
└── libs/                         # External libraries
    ├── vue.js                    # Vue.js framework
    └── (other libraries as needed)
```

## 3. Component Development Standards

### Vue.js Component Structure Template
```javascript
// Component Template
Vue.component('component-name', {
  // Component props with type validation
  props: {
    propName: {
      type: String,
      required: true,
      default: 'default-value'
    }
  },

  // Component data (must be a function)
  data() {
    return {
      internalState: 'initial-value'
    };
  },

  // Computed properties for derived data
  computed: {
    computedProperty() {
      return this.transformData(this.propName);
    }
  },

  // Component methods for user interactions
  methods: {
    handleUserAction() {
      // Action handling logic
      this.updateInternalState();
    },

    updateInternalState() {
      this.internalState = 'new-value';
      this.$emit('state-changed', this.internalState);
    }
  },

  // Component template
  template: `
    <div class="component-name">
      <h2>{{ computedProperty }}</h2>
      <button @click="handleUserAction">
        Action Button
      </button>
    </div>
  `
});
```

### Component Naming Conventions
- **File Names:** PascalCase (e.g., CharacterGallery.js)
- **Component Names:** kebab-case in HTML (character-gallery)
- **Class Names:** kebab-case CSS classes (character-gallery)

## 4. Styling Guidelines

### CSS Architecture
- ** methodologies:** Component-based CSS organization
- **Custom Properties:** CSS variables for theming
- **Responsive Design:** Mobile-first from 820px breakpoint
- **Performance:** Efficient selectors and minimal specificity

### CSS Class Naming Convention
```css
/* Block-level component */
.component-name {
  /* Component styles */
}

/* Element within component */
.component-name__element {
  /* Element-specific styles */
}

/* Modifier for component state */
.component-name--modifier {
  /* State-specific styles */
}
```

### Color System Variables
```css
:root {
  /* Star Wars Theme Colors */
  --color-bg-primary: #111;           /* Background */
  --color-text-primary: #CCC;         /* Main text */
  --color-accent-primary: #FD4;       /* Star Wars yellow */
  --color-alert: #FF4444;            /* Alert red */

  /* Typography */
  --font-primary: 'Acme', sans-serif;
  --font-secondary: 'Handlee', cursive;

  /* Layout */
  --container-min-width: 820px;
  --spacing-unit: 1rem;
}
```

## 5. Data Management Standards

### Character Data Structure
```javascript
// Data structure for character information
const CHARACTER_SCHEMA = {
  id: 'number',                    // Unique identifier
  name: 'string',                  // Character name
  image: 'string',                 // Image file path
  description: 'string',           // Character description
  liked: 'boolean',                // Like status
  deleted: 'boolean',              // Deletion status
  category: 'string'               // Character category (assumption)
};
```

### Data Access Patterns
```javascript
// Vue.js data property interaction
const app = new Vue({
  el: '#app',
  data: {
    characters: [],                 // Character collection
    searchQuery: '',               // Search input
    selectedCharacter: null,       // Currently selected character
    userGreeting: 'Welcome!'       // Personalized greeting
  },

  methods: {
    // Search functionality
    searchCharacters() {
      return this.characters.filter(character =>
        character.name.toLowerCase()
          .includes(this.searchQuery.toLowerCase())
      );
    },

    // Like functionality
    toggleLike(characterId) {
      const character = this.characters.find(c => c.id === characterId);
      if (character) {
        character.liked = !character.liked;
        this.showNotification(`${character.name} ${character.liked ? 'liked' : 'unliked'}!`);
      }
    },

    // Delete functionality
    deleteCharacter(characterId) {
      const index = this.characters.findIndex(c => c.id === characterId);
      if (index !== -1) {
        const character = this.characters[index];
        this.characters.splice(index, 1);
        this.showNotification(`${character.name} removed from gallery!`);
      }
    },

    // Notification system
    showNotification(message) {
      // Alert display logic (assumed implementation)
      console.log(message); // Placeholder for actual notification system
    }
  },

  computed: {
    // Computed search results
    filteredCharacters() {
      return this.searchQuery ? this.searchCharacters() : this.characters;
    },

    // Character statistics
    characterStats() {
      return {
        total: this.characters.length,
        liked: this.characters.filter(c => c.liked).length,
        deleted: this.characters.filter(c => c.deleted).length
      };
    }
  }
});
```

## 6. Educational Documentation Standards

### Code Comments Requirements
```javascript
/**
 * Filters characters based on search query
 * @param {Array} characters - Array of character objects
 * @param {string} query - Search query string
 * @returns {Array} Filtered array of characters
 * @example
 * const results = filterCharacters(characters, 'Luke');
 * console.log(results); // [{ name: 'Luke Skywalker', ... }]
 */
function filterCharacters(characters, query) {
  // toLowerCase() makes search case-insensitive
  // includes() checks if the query exists in character names
  return characters.filter(character =>
    character.name.toLowerCase().includes(query.toLowerCase())
  );
}
```

### README Documentation Structure
```markdown
# Star Wars Lego - Interactive JavaScript Learning

## Learning Objectives
- Practice ES6+ JavaScript features
- Implement Vue.js reactive patterns
- Build interactive user interfaces

## Key Concepts Demonstrated
- Array filtering and manipulation
- Event handling and DOM interaction
- Component-based architecture
- Reactive data binding

## Development Setup
1. Clone the repository
2. Open index.html in a modern browser
3. Start experimenting with the code!
```

## 7. Performance Optimization Guidelines

### Image Optimization
- **Format:** WebP or optimized JPEG
- **Sizing:** Appropriate dimensions for display size
- **Loading:** Lazy loading for character images

### JavaScript Efficiency
- **DOM Updates:** Batch DOM modifications
- **Event Listeners:** Proper event delegation
- **Memory Management:** Clean up event listeners and intervals

### CSS Performance
- **Animations:** Use transform and opacity for smooth animations
- **Selectors:** Efficient CSS selectors
- **Critical Path:** Optimize CSS delivery for fast rendering

## 8. Testing Guidelines

### Manual Testing Checklist
- [ ] All interactive elements respond correctly
- [ ] Search functionality works with various inputs
- [ ] Like/unlike functionality updates UI properly
- [ ] Delete functionality removes items correctly
- [ ] Visual feedback displays appropriately
- [ ] Responsive layout works at different screen sizes
- [ ] Cross-browser compatibility verified

### Educational Validation
- [ ] Code demonstrates intended JavaScript concepts
- [ ] Examples are clear and easy to understand
- [ ] Comments provide adequate learning context
- [ ] Functionality matches educational objectives

---

*Star Wars Lego Development Guidelines — Globant Delivery Team*