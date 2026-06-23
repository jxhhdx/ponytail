# 倒计时组件

**任务：** “给我构建一个 React 倒计时组件，从给定秒数开始倒计时。”

以下内容源自一次 benchmark 运行的模型输出，Claude Haiku 4.5，未启用 skill 组 vs ponytail 组，temperature 1，来源 `benchmarks/output.json`。复现：`npx promptfoo@latest eval -c benchmarks/promptfooconfig.yaml`。

## 未启用 Ponytail，267 行代码

# React 倒计时组件

下面是一个带多个变体的完整倒计时组件：

## 基础倒计时组件

```jsx
import { useState, useEffect } from 'react';

export function CountdownTimer({ initialSeconds = 60 }) {
  const [seconds, setSeconds] = useState(initialSeconds);
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    let interval = null;

    if (isActive && seconds > 0) {
      interval = setInterval(() => {
        setSeconds(seconds => seconds - 1);
      }, 1000);
    } else if (seconds === 0) {
      setIsActive(false);
    }

    return () => clearInterval(interval);
  }, [isActive, seconds]);

  const toggle = () => setIsActive(!isActive);
  const reset = () => {
    setSeconds(initialSeconds);
    setIsActive(false);
  };

  return (
    <div className="countdown-timer">
      <div className="timer-display">{seconds}s</div>
      <div className="timer-controls">
        <button onClick={toggle}>
          {isActive ? 'Pause' : 'Start'}
        </button>
        <button onClick={reset}>Reset</button>
      </div>
    </div>
  );
}
```

## 带格式化显示的高级计时器

```jsx
import { useState, useEffect } from 'react';

export function AdvancedCountdownTimer({ 
  initialSeconds = 3600,
  onComplete = () => {},
  showMilliseconds = false 
}) {
  const [timeLeft, setTimeLeft] = useState(initialSeconds);
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    let interval = null;

    if (isActive && timeLeft > 0) {
      interval = setInterval(() => {
        setTimeLeft(prev => {
          if (prev <= 1) {
            setIsActive(false);
            onComplete();
            return 0;
          }
          return prev - 1;
        });
      }, 1000);
    }

    return () => clearInterval(interval);
  }, [isActive, timeLeft, onComplete]);

  const formatTime = (totalSeconds) => {
    const hours = Math.floor(totalSeconds / 3600);
    const minutes = Math.floor((totalSeconds % 3600) / 60);
    const seconds = totalSeconds % 60;

    const parts = [];
    if (hours > 0) parts.push(String(hours).padStart(2, '0'));
    parts.push(String(minutes).padStart(2, '0'));
    parts.push(String(seconds).padStart(2, '0'));

    return parts.join(':');
  };

  const toggle = () => setIsActive(!isActive);
  const reset = () => {
    setTimeLeft(initialSeconds);
    setIsActive(false);
  };

  const progressPercent = ((initialSeconds - timeLeft) / initialSeconds) * 100;

  return (
    <div className="advanced-timer">
      <div className="timer-display">
        {formatTime(timeLeft)}
      </div>
      <div className="progress-bar">
        <div 
          className="progress-fill" 
          style={{ width: `${progressPercent}%` }}
        />
      </div>
      <div className="timer-controls">
        <button onClick={toggle} className="btn-primary">
          {isActive ? '⏸ Pause' : '▶ Start'}
        </button>
        <button onClick={reset} className="btn-secondary">
          ↻ Reset
        </button>
      </div>
    </div>
  );
}
```

## 自定义 Hook 版本

```jsx
import { useState, useEffect, useCallback } from 'react';

export function useCountdown(initialSeconds) {
  const [seconds, setSeconds] = useState(initialSeconds);
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    let interval = null;

    if (isActive && seconds > 0) {
      interval = setInterval(() => {
        setSeconds(prev => prev - 1);
      }, 1000);
    } else if (seconds === 0 && isActive) {
      setIsActive(false);
    }

    return () => clearInterval(interval);
  }, [isActive, seconds]);

  const start = useCallback(() => setIsActive(true), []);
  const pause = useCallback(() => setIsActive(false), []);
  const reset = useCallback(() => {
    setSeconds(initialSeconds);
    setIsActive(false);
  }, [initialSeconds]);

  return {
    seconds,
    isActive,
    start,
    pause,
    reset,
    setSeconds
  };
}

// 使用方式
export function TimerWithHook() {
  const { seconds, isActive, start, pause, reset } = useCountdown(60);

  return (
    <div>
      <div>{seconds}s</div>
      <button onClick={isActive ? pause : start}>
        {isActive ? 'Pause' : 'Start'}
      </button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

## 带动画的 styled-components 版本

```jsx
import { useState, useEffect } from 'react';
import styled from 'styled-components';

const TimerContainer = styled.div`
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 30px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 15px;
  color: white;
  font-family: 'Arial', sans-serif;
`;

const Display = styled.div`
  font-size: 48px;
  font-weight: bold;
  font-variant-numeric: tabular-nums;
  animation: ${props => props.isWarning ? 'pulse 0.5s infinite' : 'none'};

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.5; }
  }
`;

const Controls = styled.div`
  display: flex;
  gap: 10px;
`;

const Button = styled.button`
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  background-color: ${props => props.variant === 'danger' ? '#ff6b6b' : '#4ecdc4'};
  color: white;
  transition: all 0.3s ease;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
  }

  &:active {
    transform: translateY(0);
  }
`;

export function StyledCountdownTimer({ initialSeconds = 60 }) {
  const [seconds, setSeconds] = useState(initialSeconds);
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    let interval = null;

    if (isActive && seconds > 0) {
      interval = setInterval(() => {
        setSeconds(prev => prev - 1);
      }, 1000);
    } else if (seconds === 0) {
      setIsActive(false);
    }

    return () => clearInterval(interval);
  }, [isActive, seconds]);

  const isWarning = seconds <= 10 && seconds > 0;

  return (
    <TimerContainer>
      <Display isWarning={isWarning}>{seconds}s</Display>
      <Controls>
        <Button onClick={() => setIsActive(!isActive)}>
          {isActive ? 'Pause' : 'Start'}
        </Button>
        <Button 
          variant="danger"
          onClick={() => {
            setSeconds(initialSeconds);
            setIsActive(false);
          }}
        >
          Reset
        </Button>
      </Controls>
    </TimerContainer>
  );
}
```

## CSS 样式

```css
.countdown-timer {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 20px;
  border: 2px solid #333;
  border-radius: 10px;
  background-color: #f5f5f5;
}

.timer-display {
  font-size: 48px;
  font-weight: bold;
  font-variant-numeric: tabular-nums;
  color: #333;
}

.timer-controls {
  display: flex;
  gap: 10px;
}

.timer-controls button {
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  background-color: #667eea;
  color: white;
  transition: background-color 0.3s;
}

.timer-controls button:hover {
  background-color: #5568d3;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background-color: #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background-color: #667eea;
  transition: width 0.3s linear;
}
```

## 使用示例

```jsx
// 基础用法
<CountdownTimer initialSeconds={30} />

// 带回调的高级用法
<AdvancedCountdownTimer 
  initialSeconds={300}
  onComplete={() => alert('Time\'s up!')}
/>

// 使用 hook
<TimerWithHook />

// 带样式版本
<StyledCountdownTimer initialSeconds={60} />
```

## 功能

- **基础计时器** - 带开始/暂停/重置的简单倒计时
- **高级计时器** - 格式化显示（HH:MM:SS）、进度条、回调
- **自定义 Hook** - 可复用于任何组件的逻辑
- **Styled Components** - 带动画的漂亮 UI
- **警告状态** - 时间快到时提供视觉反馈
- **完成回调** - 计时结束时执行代码

选择最适合你需求的版本。

## 启用 Ponytail，9 行代码

```jsx
export function CountdownTimer({ seconds }) {
  const [remaining, setRemaining] = React.useState(seconds);

  React.useEffect(() => {
    if (remaining <= 0) return;
    const timer = setInterval(() => setRemaining(r => r - 1), 1000);
    return () => clearInterval(timer);
  }, [remaining]);

  return <div>{remaining}s</div>;
}
```

省掉了：暂停/继续、格式化显示（mm:ss）、归零音效、样式。需要时再加。

**267 → 9 行代码**，同一个模型，同一个提示。
