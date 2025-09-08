---
title: "Entendendo Callbacks e Tratamento de Erros: Um Caso Prático"
description: "Aprenda como fazer dados 'viajarem' entre funções usando callbacks e implementar tratamento de erros eficiente com exemplos práticos"
tags: ["javascript", "callbacks", "async", "error-handling", "tutorial"]
date: "2025-09-08"
author: "Maciel Alves"
image: "https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=800&h=400&fit=crop"
readTime: "7 min"
---

# Entendendo Callbacks e Tratamento de Erros: Um Caso Prático

## O Problema: Como Fazer Dados "Viajarem" Entre Funções

### Cenário Inicial
Imagine que você tem uma aplicação que busca livros de uma API e os exibe na tela. Você tem duas funções principais:

```javascript
// main.js - Ponto de entrada
buscarLivros(exibirOsLivrosNaTela)

// buscarLivros.js - Busca dados da API
function buscarLivros(functionPar) {
    // ... código ...
    functionPar(livrosObj)  // Chama a função passando livrosObj
}

// exibirOsLivrosNaTela.js - Exibe dados na tela
function exibirOsLivrosNaTela(livrosObj) {
    // Espera receber livrosObj como parâmetro
}
```

### A Confusão Inicial
**Pergunta comum:** "Como `exibirOsLivrosNaTela` vai receber `livrosObj` se eu não passo nenhum parâmetro para `buscarLivros`?"

**Resposta:** Você está pensando de forma **síncrona** quando deveria pensar de forma **assíncrona**.

## Lição 1: Entendendo o Padrão Callback

### O Fluxo de Dados
1. **`buscarLivros`** é responsável por **buscar** os dados da API
2. **`exibirOsLivrosNaTela`** é responsável por **exibir** os dados na tela
3. **`livrosObj`** é o **dado** que precisa "viajar" de uma função para outra

### Analogia do Cozinheiro
Imagine que você tem:
- Um **cozinheiro** (`buscarLivros`) que vai buscar ingredientes
- Uma **receita** (`exibirOsLivrosNaTela`) que precisa dos ingredientes
- Você dá a receita para o cozinheiro e diz: "quando tiver os ingredientes, execute esta receita"

### Como Funciona na Prática
```javascript
// ❌ ERRADO - Executa imediatamente
buscarLivros(exibirOsLivrosNaTela())

// ✅ CORRETO - Passa a função para ser executada depois
buscarLivros(exibirOsLivrosNaTela)
```

### O Fluxo Real
1. `main.js` chama `buscarLivros(exibirOsLivrosNaTela)`
2. `buscarLivros` recebe a função `exibirOsLivrosNaTela` como parâmetro
3. `buscarLivros` busca os dados da API
4. `buscarLivros` chama `exibirOsLivrosNaTela(livrosObj)` passando os dados
5. `exibirOsLivrosNaTela` recebe os dados e os exibe

## Lição 2: Tratamento de Erros com Callbacks

### O Problema dos Erros
```javascript
// Código atual - só loga o erro
async function buscarLivros(functionPar) {
    try {
        let livrosObj = await getBuscarLivrosDaAPI(endpointDaAPI)  
        if (livrosObj) {
            functionPar(livrosObj)  // ✅ Sucesso
        }
    } catch (error) {
        console.log(error)  // ❌ Só loga, não informa o usuário
    }
}
```

### Estratégias para Tratamento de Erros

#### Estratégia 1: Dois Callbacks (Sucesso + Erro)
```javascript
// Como usar:
buscarLivros(
    exibirOsLivrosNaTela,        // Função para sucesso
    exibirMensagemDeErro         // Função para erro
)

// Implementação:
async function buscarLivros(onSuccess, onError) {
    try {
        let livrosObj = await getBuscarLivrosDaAPI(endpointDaAPI)  
        if (livrosObj) {
            onSuccess(livrosObj)     // ✅ Sucesso
        }
    } catch (error) {
        onError(error)               // ❌ Erro
    }
}
```

**Prós:** Claro, explícito
**Contras:** Mais parâmetros, pode ficar confuso

#### Estratégia 2: Um Callback que Recebe Status
```javascript
// Como usar:
buscarLivros(exibirResultado)

// Implementação:
async function buscarLivros(callback) {
    try {
        let livrosObj = await getBuscarLivrosDaAPI(endpointDaAPI)  
        if (livrosObj) {
            callback({ success: true, data: livrosObj })
        }
    } catch (error) {
        callback({ success: false, error: error })
    }
}

// Função que processa o resultado:
function exibirResultado(resultado) {
    if (resultado.success) {
        exibirOsLivrosNaTela(resultado.data)
    } else {
        exibirMensagemDeErro(resultado.error)
    }
}
```

**Prós:** Um só callback, flexível
**Contras:** Precisa verificar o status

#### Estratégia 3: Promise (Mais Moderna)
```javascript
// Como usar:
buscarLivros()
    .then(livros => exibirOsLivrosNaTela(livros))
    .catch(erro => exibirMensagemDeErro(erro))

// Implementação:
async function buscarLivros() {
    try {
        let livrosObj = await getBuscarLivrosDaAPI(endpointDaAPI)  
        return livrosObj
    } catch (error) {
        throw error  // Re-lança o erro para ser capturado pelo .catch()
    }
}
```

**Prós:** Padrão moderno, limpo
**Contras:** Muda a forma de chamar a função

## Lição 3: Pensando Como Usuário

### Perguntas Importantes
1. **O que o usuário deve ver quando der erro?**
   - Mensagem genérica: "Erro ao carregar livros"
   - Mensagem específica: "Erro de conexão" ou "Servidor indisponível"

2. **Onde exibir a mensagem de erro?**
   - No mesmo lugar dos livros
   - Em um local específico para erros
   - Em um modal/popup

3. **O que fazer após o erro?**
   - Tentar novamente automaticamente
   - Mostrar botão "Tentar novamente"
   - Apenas informar o erro

### Exemplo Prático
```javascript
function exibirMensagemDeErro(erro) {
    const elementoLivros = document.getElementById('livros')
    elementoLivros.innerHTML = `
        <div class="erro">
            <p>Ops! Não foi possível carregar os livros.</p>
            <p>Tente novamente em alguns instantes.</p>
            <button onclick="location.reload()">Tentar Novamente</button>
        </div>
    `
}
```

## Conclusão: Padrão Mental para Callbacks

### Perguntas que Você Deve Fazer
1. **Quem tem os dados?** → `buscarLivros`
2. **Quem precisa dos dados?** → `exibirOsLivrosNaTela`
3. **Como fazer os dados viajarem?** → Passar uma função que será chamada com os dados

### Padrão Mental
```
Dados → Função que Processa → Função que Usa o Resultado
```

### Resumo do Pensamento
O tratamento de erros é sobre **comunicação**:
- **Comunicar** com o usuário quando algo dá errado
- **Comunicar** de forma clara e útil
- **Comunicar** as opções disponíveis (tentar novamente, etc.)

**A chave é pensar: "Se eu fosse o usuário, o que eu gostaria de ver quando algo desse errado?"**

---

*Este artigo demonstra como pensar sobre callbacks e tratamento de erros através de um caso prático real, mostrando diferentes estratégias e suas implicações.*
