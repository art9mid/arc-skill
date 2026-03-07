# Hook Templates

All custom hooks live in `src/hooks/`. Component-specific hooks live in their component folder.

## Global Hook Template

```typescript
// src/hooks/use-[hook-name].ts

import { useState, useCallback, useMemo } from 'react';

/**
 * [Brief description of what the hook does]
 */
export const use[HookName] = (/* params */) => {
  // State
  const [value, setValue] = useState(/* initial */);

  // Memoized callbacks
  const handler = useCallback(() => {
    // logic
  }, [/* deps */]);

  // Computed values
  const computed = useMemo(() => {
    // derived state
  }, [/* deps */]);

  return { value, handler, computed };
};
```

## Core App Hooks

### useAppTheme

```typescript
// src/hooks/use-app-theme.ts

import { useContext } from 'react';
import { AppInitializationContext } from '@/providers/app-initialization';
import type { AppTheme } from '@/theme/interfaces';

export const useAppTheme = (): AppTheme => {
  const context = useContext(AppInitializationContext);
  if (!context) {
    throw new Error('useAppTheme must be used within AppInitializationProvider');
  }
  return context.theme;
};
```

### useAppNavigation

```typescript
// src/hooks/use-app-navigation.ts

import { useNavigation } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';
import type { RootStackParamList } from '@/types/navigation.types';

export const useAppNavigation = () =>
  useNavigation<NativeStackNavigationProp<RootStackParamList>>();
```

### useStyles

```typescript
// src/hooks/use-styles.ts

import { useMemo } from 'react';
import { StyleSheet } from 'react-native';
import { useSafeAreaInsets, type EdgeInsets } from 'react-native-safe-area-context';
import { useAppTheme } from './use-app-theme';
import type { AppTheme } from '@/theme/interfaces';

type StyleCallback<T extends StyleSheet.NamedStyles<T>> = (
  theme: AppTheme,
  insets: EdgeInsets,
) => T;

export const useStyles = <T extends StyleSheet.NamedStyles<T>>(
  styleCallback: StyleCallback<T>,
) => {
  const theme = useAppTheme();
  const insets = useSafeAreaInsets();

  const styles = useMemo(
    () => StyleSheet.create(styleCallback(theme, insets)),
    [theme, insets, styleCallback],
  );

  return { styles, theme, insets };
};
```

### useAppInitialization (accessing full context)

```typescript
// src/hooks/use-app-initialization.ts

import { useContext } from 'react';
import { AppInitializationContext } from '@/providers/app-initialization';

export const useAppInitialization = () => {
  const context = useContext(AppInitializationContext);
  if (!context) {
    throw new Error(
      'useAppInitialization must be used within AppInitializationProvider',
    );
  }
  return context;
};
```

### useDebounce

```typescript
// src/hooks/use-debounce.ts

import { useState, useEffect } from 'react';

export const useDebounce = <T>(value: T, delay: number = 500): T => {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
};
```

### useKeyboard

```typescript
// src/hooks/use-keyboard.ts

import { useState, useEffect } from 'react';
import { Keyboard, KeyboardEvent, Platform } from 'react-native';

export const useKeyboard = () => {
  const [isVisible, setIsVisible] = useState(false);
  const [height, setHeight] = useState(0);

  useEffect(() => {
    const showEvent = Platform.OS === 'ios' ? 'keyboardWillShow' : 'keyboardDidShow';
    const hideEvent = Platform.OS === 'ios' ? 'keyboardWillHide' : 'keyboardDidHide';

    const showSub = Keyboard.addListener(showEvent, (e: KeyboardEvent) => {
      setIsVisible(true);
      setHeight(e.endCoordinates.height);
    });

    const hideSub = Keyboard.addListener(hideEvent, () => {
      setIsVisible(false);
      setHeight(0);
    });

    return () => {
      showSub.remove();
      hideSub.remove();
    };
  }, []);

  return { isVisible, height };
};
```

## Component-Specific Hook Template

```typescript
// src/components/[component-name]/[component-name].hooks.ts

import { useState, useCallback } from 'react';

export const use[ComponentName]State = () => {
  const [isOpen, setIsOpen] = useState(false);

  const toggle = useCallback(() => setIsOpen((prev) => !prev), []);
  const open = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);

  return { isOpen, toggle, open, close };
};
```

## Rules

1. **Hooks start with `use`** — React requirement
2. **One hook per file** for global hooks in `src/hooks/`
3. **Component-specific hooks** go in `[component].hooks.ts` in the component folder
4. **Always memoize** callbacks and values returned from hooks
5. **Throw descriptive errors** when used outside required providers
6. **Return objects, not arrays** — `{ value, handler }` not `[value, handler]`
   (exception: simple two-value hooks like `useDebounce`)
