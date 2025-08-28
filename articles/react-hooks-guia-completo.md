---
title: "React Hooks: Guia Completo para Iniciantes"
description: "Aprenda tudo sobre React Hooks: useState, useEffect, useContext e muito mais com exemplos práticos"
tags: ["react", "hooks", "javascript", "frontend", "tutorial"]
date: "2024-01-20"
author: "Maciel Alves"
image: "https://images.unsplash.com/photo-1633356122544-f134324a6cee?w=800&h=400&fit=crop"
readTime: "8 min"
---

# React Hooks: Guia Completo para Iniciantes

## Introdução

React Hooks revolucionaram a forma como escrevemos componentes funcionais. Neste guia completo, vou explicar os principais hooks e como usá-los de forma eficiente.

## O que são React Hooks?

Hooks são funções especiais que permitem usar estado e outros recursos do React em componentes funcionais. Eles foram introduzidos no React 16.8 e desde então se tornaram a forma padrão de escrever componentes.

## useState - Gerenciando Estado

O `useState` é o hook mais básico e fundamental:

```javascript
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Você clicou {count} vezes</p>
      <button onClick={() => setCount(count + 1)}>
        Clique aqui
      </button>
    </div>
  );
}
```

### Dicas importantes:

- **Sempre use o setter**: Nunca modifique o estado diretamente
- **Múltiplos estados**: Você pode usar useState várias vezes
- **Estado inicial**: Pode ser uma função para cálculos complexos

## useEffect - Efeitos Colaterais

O `useState` é usado para efeitos colaterais como chamadas de API, subscriptions, etc:

```javascript
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        console.error('Erro ao buscar usuário:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]); // Dependência

  if (loading) return <div>Carregando...</div>;
  if (!user) return <div>Usuário não encontrado</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Tipos de useEffect:

1. **Sem dependências**: Executa a cada render
2. **Com array vazio**: Executa apenas uma vez
3. **Com dependências**: Executa quando dependências mudam

## useContext - Compartilhando Estado

Para compartilhar estado entre componentes sem prop drilling:

```javascript
import React, { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme deve ser usado dentro de ThemeProvider');
  }
  return context;
}
```

## useRef - Referências

Para acessar elementos DOM ou valores que não causam re-render:

```javascript
import React, { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} type="text" />;
}
```

## Hooks Customizados

Crie seus próprios hooks para reutilizar lógica:

```javascript
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Uso:
function MyComponent() {
  const [name, setName] = useLocalStorage('name', '');
  
  return (
    <input
      value={name}
      onChange={(e) => setName(e.target.value)}
      placeholder="Digite seu nome"
    />
  );
}
```

## Regras dos Hooks

1. **Sempre no topo**: Hooks devem ser chamados no nível superior
2. **Apenas em componentes funcionais**: Não use em classes ou funções regulares
3. **Ordem consistente**: Hooks devem ser chamados na mesma ordem sempre

## Conclusão

React Hooks tornaram o React mais simples e poderoso. Com prática e entendimento das regras, você pode criar componentes mais limpos e reutilizáveis.

---

*Este artigo faz parte da série de tutoriais sobre React no meu blog.*
