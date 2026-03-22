# Testing Guide - Two-Factor Authentication Component

## Overview

This guide covers comprehensive testing strategies for the TwoFactorAuth component, including unit tests, integration tests, accessibility testing, and manual testing procedures.

## Test Setup

### Dependencies

```json
{
  "devDependencies": {
    "@testing-library/svelte": "^4.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/user-event": "^14.0.0",
    "vitest": "^1.0.0",
    "jsdom": "^23.0.0",
    "playwright": "^1.40.0"
  }
}
```

### Configuration

#### Vitest Config (`vite.config.js`)

```javascript
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vitest/config';

export default defineConfig({
  plugins: [sveltekit()],
  test: {
    include: ['src/**/*.{test,spec}.{js,ts}'],
    environment: 'jsdom',
    setupFiles: ['./src/test-setup.js'],
    coverage: {
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test-setup.js',
        '**/*.test.{js,ts}',
        '**/*.spec.{js,ts}'
      ]
    }
  }
});
```

#### Test Setup (`src/test-setup.js`)

```javascript
import '@testing-library/jest-dom';
import { vi } from 'vitest';

// Mock clipboard API
Object.assign(navigator, {
  clipboard: {
    readText: vi.fn(() => Promise.resolve('123456')),
    writeText: vi.fn(() => Promise.resolve())
  }
});

// Mock IntersectionObserver
global.IntersectionObserver = vi.fn(() => ({
  observe: vi.fn(),
  disconnect: vi.fn(),
  unobserve: vi.fn()
}));

// Suppress console.error for expected test errors
const originalError = console.error;
beforeAll(() => {
  console.error = (...args) => {
    if (
      typeof args[0] === 'string' &&
      args[0].includes('Warning: ReactDOM.render is no longer supported')
    ) {
      return;
    }
    originalError.call(console, ...args);
  };
});

afterAll(() => {
  console.error = originalError;
});
```

## Unit Tests

### Basic Functionality Tests

```javascript
// src/lib/components/TwoFactorAuth.test.js
import { render, screen } from '@testing-library/svelte';
import userEvent from '@testing-library/user-event';
import { vi } from 'vitest';
import TwoFactorAuth from './TwoFactorAuth.svelte';

describe('TwoFactorAuth Component', () => {
  test('renders with correct initial state', () => {
    render(TwoFactorAuth);
    
    // Check for 6 input fields
    const inputs = screen.getAllByRole('textbox');
    expect(inputs).toHaveLength(6);
    
    // Check initial placeholder text
    expect(screen.getByText('Enter 6-digit code...')).toBeInTheDocument();
    
    // Check first input has focus
    expect(inputs[0]).toHaveFocus();
  });

  test('accepts valid numeric input', async () => {
    const user = userEvent.setup();
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    // Type a digit
    await user.type(inputs[0], '1');
    
    expect(inputs[0]).toHaveValue('1');
    expect(inputs[1]).toHaveFocus();
  });

  test('rejects non-numeric input', async () => {
    const user = userEvent.setup();
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    // Try to type a letter
    await user.type(inputs[0], 'a');
    
    expect(inputs[0]).toHaveValue('');
    expect(inputs[0]).toHaveFocus();
  });

  test('handles backspace correctly', async () => {
    const user = userEvent.setup();
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    // Type digits in first two fields
    await user.type(inputs[0], '1');
    await user.type(inputs[1], '2');
    
    // Press backspace
    await user.keyboard('{Backspace}');
    
    expect(inputs[1]).toHaveValue('');
    expect(inputs[1]).toHaveFocus();
    
    // Press backspace again
    await user.keyboard('{Backspace}');
    
    expect(inputs[0]).toHaveValue('');
    expect(inputs[0]).toHaveFocus();
  });

  test('handles arrow key navigation', async () => {
    const user = userEvent.setup();
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    // Start at first input
    inputs[0].focus();
    
    // Arrow right
    await user.keyboard('{ArrowRight}');
    expect(inputs[1]).toHaveFocus();
    
    // Arrow left
    await user.keyboard('{ArrowLeft}');
    expect(inputs[0]).toHaveFocus();
  });

  test('auto-submits when all fields are filled', async () => {
    const user = userEvent.setup();
    const onSuccess = vi.fn();
    
    render(TwoFactorAuth, {
      props: {
        correctCode: '123456',
        on: { success: onSuccess }
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Fill all inputs
    for (let i = 0; i < 6; i++) {
      await user.type(inputs[i], String(i + 1));
    }
    
    // Wait for verification
    await vi.waitFor(() => {
      expect(onSuccess).toHaveBeenCalledWith(
        expect.objectContaining({
          detail: { code: '123456' }
        })
      );
    });
  });

  test('shows error state for incorrect code', async () => {
    const user = userEvent.setup();
    const onError = vi.fn();
    
    render(TwoFactorAuth, {
      props: {
        correctCode: '123456',
        on: { error: onError }
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Fill with incorrect code
    await user.type(inputs[0], '1');
    await user.type(inputs[1], '1');
    await user.type(inputs[2], '1');
    await user.type(inputs[3], '1');
    await user.type(inputs[4], '1');
    await user.type(inputs[5], '1');
    
    await vi.waitFor(() => {
      expect(onError).toHaveBeenCalled();
      expect(screen.getByText('Invalid code')).toBeInTheDocument();
    });
  });
});
```

### Paste Functionality Tests

```javascript
describe('Paste Functionality', () => {
  test('handles Ctrl+V paste', async () => {
    const user = userEvent.setup();
    
    // Mock clipboard
    vi.spyOn(navigator.clipboard, 'readText').mockResolvedValue('123456');
    
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    // Focus first input and paste
    inputs[0].focus();
    await user.keyboard('{Control>}v{/Control}');
    
    // Check all inputs are filled
    expect(inputs[0]).toHaveValue('1');
    expect(inputs[1]).toHaveValue('2');
    expect(inputs[2]).toHaveValue('3');
    expect(inputs[3]).toHaveValue('4');
    expect(inputs[4]).toHaveValue('5');
    expect(inputs[5]).toHaveValue('6');
  });

  test('handles right-click paste', async () => {
    const user = userEvent.setup();
    
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    // Simulate paste event
    await user.click(inputs[0], { button: 2 }); // Right click
    
    const pasteEvent = new ClipboardEvent('paste', {
      clipboardData: new DataTransfer()
    });
    pasteEvent.clipboardData.setData('text/plain', '654321');
    
    inputs[0].dispatchEvent(pasteEvent);
    
    // Check inputs are filled
    expect(inputs[0]).toHaveValue('6');
    expect(inputs[1]).toHaveValue('5');
    expect(inputs[2]).toHaveValue('4');
    expect(inputs[3]).toHaveValue('3');
    expect(inputs[4]).toHaveValue('2');
    expect(inputs[5]).toHaveValue('1');
  });

  test('filters non-numeric characters from paste', async () => {
    const user = userEvent.setup();
    
    vi.spyOn(navigator.clipboard, 'readText').mockResolvedValue('1a2b3c4d5e6f');
    
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    inputs[0].focus();
    
    await user.keyboard('{Control>}v{/Control}');
    
    // Should only use numeric characters
    expect(inputs[0]).toHaveValue('1');
    expect(inputs[1]).toHaveValue('2');
    expect(inputs[2]).toHaveValue('3');
    expect(inputs[3]).toHaveValue('4');
    expect(inputs[4]).toHaveValue('5');
    expect(inputs[5]).toHaveValue('6');
  });
});
```

### Component Methods Tests

```javascript
describe('Component Methods', () => {
  test('reset() clears all inputs and resets state', async () => {
    const user = userEvent.setup();
    let component;
    
    render(TwoFactorAuth, {
      props: {
        bind: { this: component }
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Fill some inputs
    await user.type(inputs[0], '1');
    await user.type(inputs[1], '2');
    
    // Reset
    component.reset();
    
    // Check all inputs are empty
    inputs.forEach(input => {
      expect(input).toHaveValue('');
    });
    
    // Check focus is on first input
    expect(inputs[0]).toHaveFocus();
  });

  test('setValue() sets the code programmatically', () => {
    let component;
    
    render(TwoFactorAuth, {
      props: {
        bind: { this: component }
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Set value
    component.setValue('789012');
    
    // Check inputs
    expect(inputs[0]).toHaveValue('7');
    expect(inputs[1]).toHaveValue('8');
    expect(inputs[2]).toHaveValue('9');
    expect(inputs[3]).toHaveValue('0');
    expect(inputs[4]).toHaveValue('1');
    expect(inputs[5]).toHaveValue('2');
  });

  test('getCurrentCode() returns current code', async () => {
    const user = userEvent.setup();
    let component;
    
    render(TwoFactorAuth, {
      props: {
        bind: { this: component }
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Type partial code
    await user.type(inputs[0], '1');
    await user.type(inputs[1], '2');
    await user.type(inputs[2], '3');
    
    expect(component.getCurrentCode()).toBe('123   ');
  });
});
```

## Integration Tests

### API Integration Tests

```javascript
// src/tests/integration/api-integration.test.js
import { render, screen } from '@testing-library/svelte';
import userEvent from '@testing-library/user-event';
import { vi } from 'vitest';
import TwoFactorAuth from '$lib/components/TwoFactorAuth.svelte';

describe('API Integration', () => {
  test('integrates with custom verification function', async () => {
    const user = userEvent.setup();
    const mockVerify = vi.fn().mockResolvedValue(true);
    const onSuccess = vi.fn();
    
    render(TwoFactorAuth, {
      props: {
        verifyCode: mockVerify,
        on: { success: onSuccess }
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Fill code
    for (let i = 0; i < 6; i++) {
      await user.type(inputs[i], '1');
    }
    
    // Wait for verification
    await vi.waitFor(() => {
      expect(mockVerify).toHaveBeenCalledWith('111111');
      expect(onSuccess).toHaveBeenCalled();
    });
  });

  test('handles API errors gracefully', async () => {
    const user = userEvent.setup();
    const mockVerify = vi.fn().mockRejectedValue(new Error('Network error'));
    const onError = vi.fn();
    
    render(TwoFactorAuth, {
      props: {
        verifyCode: mockVerify,
        on: { error: onError }
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Fill code
    for (let i = 0; i < 6; i++) {
      await user.type(inputs[i], '1');
    }
    
    await vi.waitFor(() => {
      expect(onError).toHaveBeenCalled();
      expect(screen.getByText('Invalid code')).toBeInTheDocument();
    });
  });

  test('handles rate limiting', async () => {
    const user = userEvent.setup();
    let attemptCount = 0;
    
    const mockVerify = vi.fn().mockImplementation(() => {
      attemptCount++;
      return Promise.resolve(false);
    });
    
    render(TwoFactorAuth, {
      props: {
        verifyCode: mockVerify,
        maxAttempts: 3
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Make 3 failed attempts
    for (let attempt = 0; attempt < 3; attempt++) {
      // Clear inputs
      inputs.forEach(input => input.value = '');
      
      // Fill code
      for (let i = 0; i < 6; i++) {
        await user.type(inputs[i], '1');
      }
      
      await vi.waitFor(() => {
        expect(mockVerify).toHaveBeenCalledTimes(attempt + 1);
      });
    }
    
    // Component should be locked
    expect(screen.getByText(/maximum attempts/i)).toBeInTheDocument();
    inputs.forEach(input => {
      expect(input).toBeDisabled();
    });
  });
});
```

## Accessibility Tests

### Screen Reader Tests

```javascript
// src/tests/accessibility/screen-reader.test.js
import { render, screen } from '@testing-library/svelte';
import userEvent from '@testing-library/user-event';
import TwoFactorAuth from '$lib/components/TwoFactorAuth.svelte';

describe('Screen Reader Accessibility', () => {
  test('has proper ARIA labels', () => {
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    inputs.forEach((input, index) => {
      expect(input).toHaveAttribute('aria-label', `Digit ${index + 1} of 6`);
      expect(input).toHaveAttribute('aria-describedby', '2fa-instructions');
    });
  });

  test('announces state changes', async () => {
    const user = userEvent.setup();
    render(TwoFactorAuth, { props: { correctCode: '123456' } });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Fill incorrect code
    for (let i = 0; i < 6; i++) {
      await user.type(inputs[i], '1');
    }
    
    await vi.waitFor(() => {
      const announcement = screen.getByRole('status');
      expect(announcement).toHaveTextContent('Invalid code');
    });
  });

  test('has proper focus management', async () => {
    const user = userEvent.setup();
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    // Tab navigation
    await user.tab();
    expect(inputs[0]).toHaveFocus();
    
    await user.tab();
    expect(inputs[1]).toHaveFocus();
    
    // Shift+Tab reverse navigation
    await user.tab({ shift: true });
    expect(inputs[0]).toHaveFocus();
  });

  test('supports high contrast mode', () => {
    // Mock high contrast media query
    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: vi.fn().mockImplementation(query => ({
        matches: query === '(prefers-contrast: high)',
        media: query,
        onchange: null,
        addListener: vi.fn(),
        removeListener: vi.fn(),
        addEventListener: vi.fn(),
        removeEventListener: vi.fn(),
        dispatchEvent: vi.fn(),
      })),
    });
    
    render(TwoFactorAuth);
    
    const container = screen.getByRole('region');
    expect(container).toHaveClass('high-contrast');
  });
});
```

### Keyboard Navigation Tests

```javascript
describe('Keyboard Navigation', () => {
  test('supports all keyboard shortcuts', async () => {
    const user = userEvent.setup();
    render(TwoFactorAuth);
    
    const inputs = screen.getAllByRole('textbox');
    
    // Test all arrow keys
    inputs[0].focus();
    await user.keyboard('{ArrowRight}');
    expect(inputs[1]).toHaveFocus();
    
    await user.keyboard('{ArrowDown}');
    expect(inputs[1]).toHaveFocus(); // Should stay on same input
    
    await user.keyboard('{ArrowLeft}');
    expect(inputs[0]).toHaveFocus();
    
    await user.keyboard('{ArrowUp}');
    expect(inputs[0]).toHaveFocus(); // Should stay on same input
    
    // Test Home/End keys
    inputs[3].focus();
    await user.keyboard('{Home}');
    expect(inputs[0]).toHaveFocus();
    
    await user.keyboard('{End}');
    expect(inputs[5]).toHaveFocus();
    
    // Test Escape key
    await user.type(inputs[0], '123');
    await user.keyboard('{Escape}');
    
    inputs.forEach(input => {
      expect(input).toHaveValue('');
    });
  });

  test('handles Enter key for submission', async () => {
    const user = userEvent.setup();
    const onSuccess = vi.fn();
    
    render(TwoFactorAuth, {
      props: {
        correctCode: '123456',
        autoSubmit: false,
        on: { success: onSuccess }
      }
    });
    
    const inputs = screen.getAllByRole('textbox');
    
    // Fill code
    await user.type(inputs[0], '123456');
    
    // Press Enter
    await user.keyboard('{Enter}');
    
    await vi.waitFor(() => {
      expect(onSuccess).toHaveBeenCalled();
    });
  });
});
```

## Visual Regression Tests

### Playwright Visual Tests

```javascript
// tests/visual/two-factor-auth.spec.js
import { test, expect } from '@playwright/test';

test.describe('TwoFactorAuth Visual Tests', () => {
  test('default state screenshot', async ({ page }) => {
    await page.goto('/');
    
    const component = page.locator('[data-testid="two-factor-auth"]');
    await expect(component).toBeVisible();
    
    await expect(component).toHaveScreenshot('default-state.png');
  });

  test('filled state screenshot', async ({ page }) => {
    await page.goto('/');
    
    const inputs = page.locator('input[type="text"]');
    
    // Fill inputs
    for (let i = 0; i < 6; i++) {
      await inputs.nth(i).fill(String(i + 1));
    }
    
    const component = page.locator('[data-testid="two-factor-auth"]');
    await expect(component).toHaveScreenshot('filled-state.png');
  });

  test('success state screenshot', async ({ page }) => {
    await page.goto('/');
    
    const inputs = page.locator('input[type="text"]');
    
    // Fill with correct code
    for (let i = 0; i < 6; i++) {
      await inputs.nth(i).fill(String(i + 1));
    }
    
    // Wait for success state
    await page.waitForSelector('[data-state="success"]');
    
    const component = page.locator('[data-testid="two-factor-auth"]');
    await expect(component).toHaveScreenshot('success-state.png');
  });

  test('error state screenshot', async ({ page }) => {
    await page.goto('/');
    
    const inputs = page.locator('input[type="text"]');
    
    // Fill with incorrect code
    for (let i = 0; i < 6; i++) {
      await inputs.nth(i).fill('1');
    }
    
    // Wait for error state
    await page.waitForSelector('[data-state="error"]');
    
    const component = page.locator('[data-testid="two-factor-auth"]');
    await expect(component).toHaveScreenshot('error-state.png');
  });

  test('mobile viewport screenshot', async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto('/');
    
    const component = page.locator('[data-testid="two-factor-auth"]');
    await expect(component).toHaveScreenshot('mobile-view.png');
  });
});
```

## Performance Tests

### Load Testing

```javascript
// tests/performance/load.test.js
import { test, expect } from '@playwright/test';

test.describe('Performance Tests', () => {
  test('component loads within performance budget', async ({ page }) => {
    await page.goto('/', { waitUntil: 'networkidle' });
    
    const performanceEntries = await page.evaluate(() => {
      return JSON.stringify(performance.getEntriesByType('navigation'));
    });
    
    const navigation = JSON.parse(performanceEntries)[0];
    
    // Check load times
    expect(navigation.loadEventEnd - navigation.fetchStart).toBeLessThan(3000);
    expect(navigation.domContentLoadedEventEnd - navigation.fetchStart).toBeLessThan(1500);
  });

  test('animations run at 60fps', async ({ page }) => {
    await page.goto('/');
    
    // Start performance monitoring
    await page.evaluate(() => {
      window.frameRates = [];
      let lastTime = performance.now();
      
      function measureFPS() {
        const currentTime = performance.now();
        const fps = 1000 / (currentTime - lastTime);
        window.frameRates.push(fps);
        lastTime = currentTime;
        
        if (window.frameRates.length < 60) {
          requestAnimationFrame(measureFPS);
        }
      }
      
      requestAnimationFrame(measureFPS);
    });
    
    // Trigger animations
    const inputs = page.locator('input[type="text"]');
    await inputs.first().fill('1');
    
    // Wait for animation
    await page.waitForTimeout(1000);
    
    const frameRates = await page.evaluate(() => window.frameRates);
    const averageFPS = frameRates.reduce((sum, fps) => sum + fps, 0) / frameRates.length;
    
    expect(averageFPS).toBeGreaterThan(55); // Allow for some variance
  });
});
```

## Manual Testing Checklist

### Functional Testing

#### Input Validation
- [ ] Accepts digits 0-9
- [ ] Rejects letters and special characters
- [ ] Handles international number formats
- [ ] Validates input length (6 digits only)

#### Navigation
- [ ] Tab key moves between inputs
- [ ] Shift+Tab moves backwards
- [ ] Arrow keys navigate correctly
- [ ] Home/End keys work
- [ ] Focus visible at all times

#### Paste Functionality
- [ ] Ctrl+V works from clipboard
- [ ] Right-click paste works
- [ ] Filters non-numeric characters
- [ ] Handles partial pastes correctly
- [ ] Works with different clipboard formats

#### Auto-advance/Submit
- [ ] Auto-advances on digit entry
- [ ] Auto-submits when complete
- [ ] Can disable auto-submit
- [ ] Enter key submits manually

#### Error Handling
- [ ] Shows error for incorrect codes
- [ ] Clears error on new input
- [ ] Handles network errors gracefully
- [ ] Respects maximum attempts

#### States and Animations
- [ ] Loading state during verification
- [ ] Success state with animation
- [ ] Error state with shake animation
- [ ] Pulse animations work correctly
- [ ] Cursor animations are smooth

### Accessibility Testing

#### Keyboard Navigation
- [ ] All functionality available via keyboard
- [ ] Logical tab order
- [ ] Focus indicators visible
- [ ] No keyboard traps

#### Screen Readers
- [ ] Proper ARIA labels
- [ ] Status announcements
- [ ] Instructions read correctly
- [ ] Error messages announced

#### Visual Accessibility
- [ ] High contrast mode support
- [ ] Sufficient color contrast (4.5:1)
- [ ] Text scales to 200%
- [ ] Works without color alone

### Browser Testing

#### Desktop Browsers
- [ ] Chrome (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest)
- [ ] Edge (latest)

#### Mobile Browsers
- [ ] iOS Safari
- [ ] Android Chrome
- [ ] Samsung Internet
- [ ] Firefox Mobile

### Device Testing

#### Screen Sizes
- [ ] Mobile (320px+)
- [ ] Tablet (768px+)
- [ ] Desktop (1024px+)
- [ ] Large screens (1440px+)

#### Input Methods
- [ ] Touch input on mobile
- [ ] Mouse input on desktop
- [ ] Keyboard-only navigation
- [ ] Voice input (if applicable)

### Performance Testing

#### Load Performance
- [ ] Initial load < 3 seconds
- [ ] First paint < 1 second
- [ ] Interactive < 2 seconds
- [ ] No layout shifts

#### Runtime Performance
- [ ] Smooth animations (60fps)
- [ ] No memory leaks
- [ ] Efficient DOM updates
- [ ] Responsive to input

## Test Commands

### Running Tests

```bash
# Run all tests
npm test

# Run unit tests only
npm run test:unit

# Run integration tests
npm run test:integration

# Run accessibility tests
npm run test:a11y

# Run visual regression tests
npm run test:visual

# Run performance tests
npm run test:performance

# Run tests with coverage
npm run test:coverage

# Watch mode for development
npm run test:watch

# Debug mode
npm run test:debug
```

### CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Test Suite
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - run: npm ci
      - run: npm run test:coverage
      - run: npm run test:a11y
      - run: npm run test:visual
      
      - uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
```

## Test Coverage Goals

- **Unit Tests**: 90%+ line coverage
- **Integration Tests**: All API interactions
- **Accessibility Tests**: WCAG 2.1 AA compliance
- **Visual Tests**: All component states
- **Performance Tests**: Core web vitals

## Debugging Tests

### Common Issues

1. **Timing Issues**: Use `waitFor` for async operations
2. **DOM Cleanup**: Ensure proper cleanup between tests
3. **Mock Issues**: Reset mocks between tests
4. **Focus Issues**: Account for focus management
5. **Animation Issues**: Consider reducing motion for tests

### Debug Tools

```javascript
// Debug component state
screen.debug();

// Debug specific element
screen.debug(screen.getByRole('textbox'));

// Log events
userEvent.setup({ delay: null, advanceTimers: vi.advanceTimersByTime });
```

This comprehensive testing guide ensures the TwoFactorAuth component is thoroughly tested for functionality, accessibility, performance, and visual consistency across all supported browsers and devices.
