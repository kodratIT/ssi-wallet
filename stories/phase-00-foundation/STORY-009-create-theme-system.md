# STORY-009: Create Theme System

**Phase**: 0 - Foundation  
**Story**: 009 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Design System** - Consistent UI across app
2. **Theme Management** - Colors, spacing, typography
3. **Dark Mode Support** - Theme switching
4. **React Context** - Share theme globally

### Mengapa Penting?

Theme system = **Consistent, professional UI** + easy maintenance!

**Analogy**: Theme = Brand guidelines - consistent colors, fonts, spacing everywhere.

---

## 🎯 Objectives

1. ✅ Create theme definition
2. ✅ Setup theme context
3. ✅ Create theme hook
4. ✅ Add dark mode support

---

## 📝 Implementation

### Create Theme Definition

Create `src/constants/theme.ts`:

```typescript
export const lightTheme = {
  colors: {
    primary: '#3B82F6',
    secondary: '#10B981',
    background: '#FFFFFF',
    card: '#F9FAFB',
    text: '#111827',
    textSecondary: '#6B7280',
    border: '#E5E7EB',
    error: '#EF4444',
    success: '#10B981',
    warning: '#F59E0B',
  },
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
  },
  typography: {
    h1: { fontSize: 32, fontWeight: 'bold' },
    h2: { fontSize: 24, fontWeight: 'bold' },
    h3: { fontSize: 20, fontWeight: '600' },
    body: { fontSize: 16, fontWeight: 'normal' },
    caption: { fontSize: 12, fontWeight: 'normal' },
  },
};

export const darkTheme = {
  ...lightTheme,
  colors: {
    ...lightTheme.colors,
    background: '#111827',
    card: '#1F2937',
    text: '#F9FAFB',
    textSecondary: '#9CA3AF',
    border: '#374151',
  },
};

export type Theme = typeof lightTheme;
```

### Create Theme Context

Create `src/contexts/ThemeContext.tsx`:

```typescript
import React, { createContext, useState, useContext, ReactNode } from 'react';
import { lightTheme, darkTheme, Theme } from '@constants/theme';

type ThemeContextType = {
  theme: Theme;
  isDark: boolean;
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider = ({ children }: { children: ReactNode }) => {
  const [isDark, setIsDark] = useState(false);
  const theme = isDark ? darkTheme : lightTheme;

  const toggleTheme = () => setIsDark(!isDark);

  return (
    <ThemeContext.Provider value={{ theme, isDark, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};
```

### Update App.tsx

```typescript
import { ThemeProvider } from '@contexts/ThemeContext';

export default function App() {
  return (
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <ThemeProvider>
          <NavigationContainer>
            <RootNavigator />
          </NavigationContainer>
        </ThemeProvider>
      </PersistGate>
    </Provider>
  );
}
```

---

## ✅ Verification

```bash
# Use theme in component:
const { theme, toggleTheme } = useTheme();

# Should have access to theme colors, spacing, etc
```

---

## 🎓 Learning

- Design system concepts
- React Context for global state
- Theme management patterns
- Dark mode implementation

**Status**: Ready | **Next**: STORY-010 - Environment Config
