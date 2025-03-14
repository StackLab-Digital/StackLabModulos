# Guia de Customização do Chatwoot via Dashboard Scripts

Este guia demonstra como personalizar os principais componentes do frontend do Chatwoot usando Dashboard Scripts. Os scripts são injetados no final da tag `<body>` e permitem modificar a aparência e comportamento do dashboard.

## Sumário

1. [Introdução](#introdução)
2. [Componentes Principais](#componentes-principais)
3. [Exemplos Práticos](#exemplos-práticos)
   - [Personalização do Kanban](#personalização-do-kanban)
   - [Customização do Sidebar](#customização-do-sidebar)
   - [Melhoria de Lista de Conversas](#melhoria-de-lista-de-conversas)
   - [Atalhos e Automações](#atalhos-e-automações)
4. [Arquitetura Vue e Estado Global](#arquitetura-vue-e-estado-global)
5. [Depuração e Desenvolvimento](#depuração-e-desenvolvimento)

## Introdução

Dashboard Scripts são fragmentos de HTML/JavaScript injetados no final de cada página do dashboard. Eles permitem customizar a aparência e comportamento da aplicação sem modificar o código-fonte original.

### Melhor Prática para Injeção

```javascript
<script>
document.addEventListener('DOMContentLoaded', function() {
  // Garantir que a aplicação Vue está completamente carregada
  const waitForVue = setInterval(() => {
    if (window.__vue__) {
      clearInterval(waitForVue);
      // Seu código aqui
      console.log('Dashboard script carregado e Vue detectado');

      customizeApp();
    }
  }, 500);

  function customizeApp() {
    // Código de customização aqui
  }
});
</script>
```

## Componentes Principais

### Estrutura de Componentes

- **Sidebar.vue**: Navegação lateral principal e secundária
- **ChatList.vue**: Lista de conversas na visualização principal
- **ConversationItem.vue**: Item individual na lista de conversas
- **KanbanItem.vue**: Item individual no quadro Kanban
- **KanbanHeader.vue**: Cabeçalho do quadro Kanban com controles e filtros
- **KanbanColumn.vue**: Coluna individual no quadro Kanban
- **KanbanActions.vue**: Ações disponíveis para itens do Kanban

## Exemplos Práticos

### Personalização do Kanban

#### Mudar o estilo dos cards do Kanban

```javascript
function customizeKanbanItems() {
  // Seletor para items do Kanban
  const kanbanItemSelector = '.kanban-item';

  // Observer para detectar novos items sendo adicionados
  const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      if (mutation.addedNodes.length) {
        document.querySelectorAll(kanbanItemSelector).forEach(item => {
          // Aplicar estilo personalizado
          item.style.borderRadius = '12px';
          item.style.boxShadow = '0 4px 8px rgba(0, 0, 0, 0.1)';

          // Adicionar borda baseada na prioridade
          const priorityEl = item.querySelector('[data-priority]');
          if (priorityEl) {
            const priority = priorityEl.getAttribute('data-priority');
            const borderColors = {
              urgent: '#7C3AED',
              high: '#EF4444',
              medium: '#F59E0B',
              low: '#10B981',
              none: '#64748B',
            };
            item.style.borderLeft = `4px solid ${borderColors[priority] || borderColors.none}`;
          }
        });
      }
    });
  });

  // Observar o container do Kanban para mudanças
  const kanbanContainer = document.querySelector('.kanban-container');
  if (kanbanContainer) {
    observer.observe(kanbanContainer, { childList: true, subtree: true });
  }
}
```

#### Adicionar funcionalidades ao header do Kanban

```javascript
function enhanceKanbanHeader() {
  const headerSelector = '.kanban-header';

  const addExportButton = () => {
    const header = document.querySelector(headerSelector);
    if (!header || header.querySelector('.custom-export-btn')) return;

    const exportBtn = document.createElement('button');
    exportBtn.className =
      'custom-export-btn woot-button woot-button--sm woot-button--primary ml-2';
    exportBtn.innerHTML = '<i class="icon-download"></i> Exportar CSV';
    exportBtn.addEventListener('click', () => {
      // Acessar dados via API Vue
      const vueInstance = window.__vue__;
      const store = vueInstance.$store;
      const kanbanItems = store.getters['kanban/getKanbanItems'] || [];

      // Gerar CSV
      const csvContent = generateCSV(kanbanItems);
      downloadCSV(csvContent, 'kanban_data.csv');
    });

    // Adicionar à DOM
    const rightSide = header.querySelector('.header-right');
    if (rightSide) rightSide.appendChild(exportBtn);
  };

  // Helper para gerar CSV a partir dos dados
  function generateCSV(items) {
    // Implementação da geração de CSV
    const headers = ['ID', 'Título', 'Etapa', 'Prioridade', 'Valor', 'Status'];
    const rows = items.map(item => [
      item.id,
      item.item_details?.title || '',
      item.funnel_stage || '',
      item.item_details?.priority || '',
      item.item_details?.value || '',
      item.item_details?.status || '',
    ]);

    return [headers, ...rows].map(row => row.join(',')).join('\\n');
  }

  // Helper para download do CSV
  function downloadCSV(content, filename) {
    const blob = new Blob([content], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    const url = URL.createObjectURL(blob);
    link.href = url;
    link.setAttribute('download', filename);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }

  // Observer para detectar quando o header é carregado
  const observer = new MutationObserver(mutations => {
    if (document.querySelector(headerSelector)) {
      addExportButton();
    }
  });

  observer.observe(document.body, { childList: true, subtree: true });
}
```

### Customização do Sidebar

```javascript
function customizeSidebar() {
  const primarySidebarSelector = '.primary-sidebar';
  const secondarySidebarSelector = '.secondary-sidebar';

  // Adicionar novo item ao menu principal
  const addCustomMenuItem = () => {
    const sidebar = document.querySelector(primarySidebarSelector);
    if (!sidebar || sidebar.querySelector('.custom-menu-item')) return;

    const menuItem = document.createElement('div');
    menuItem.className = 'custom-menu-item sidebar-menu-item';
    menuItem.innerHTML = `
      <a href="#" class="sidebar-menu-item__link">
        <span class="sidebar-menu-item__icon">
          <i class="icon-calendar"></i>
        </span>
        <span class="sidebar-menu-item__title">Agenda</span>
      </a>
    `;

    menuItem.addEventListener('click', e => {
      e.preventDefault();
      // Implementar ação desejada
      alert('Calendário personalizado');
    });

    // Inserir após o último item do menu
    const menuItems = sidebar.querySelectorAll('.sidebar-menu-item');
    const lastItem = menuItems[menuItems.length - 1];
    lastItem.parentNode.insertBefore(menuItem, lastItem.nextSibling);
  };

  // Alterar cores do sidebar
  const customizeSidebarColors = () => {
    const applyStyles = (element, styles) => {
      if (!element) return;
      Object.keys(styles).forEach(prop => {
        element.style[prop] = styles[prop];
      });
    };

    // Primary sidebar
    const primarySidebar = document.querySelector(primarySidebarSelector);
    applyStyles(primarySidebar, {
      backgroundColor: '#1e1e2d',
      color: '#ffffff',
    });

    // Secondary sidebar
    const secondarySidebar = document.querySelector(secondarySidebarSelector);
    applyStyles(secondarySidebar, {
      backgroundColor: '#252536',
      color: '#ffffff',
    });

    // Aplicar estilos aos links
    document
      .querySelectorAll(
        `${primarySidebarSelector} a, ${secondarySidebarSelector} a`
      )
      .forEach(link => {
        link.style.color = '#a5a5b9';
        link.addEventListener('mouseover', () => {
          link.style.color = '#ffffff';
        });
        link.addEventListener('mouseout', () => {
          link.style.color = '#a5a5b9';
        });
      });
  };

  // Observer para detectar quando o sidebar é carregado
  const observer = new MutationObserver(mutations => {
    if (document.querySelector(primarySidebarSelector)) {
      addCustomMenuItem();
      customizeSidebarColors();
      observer.disconnect(); // Desconectar após aplicar as mudanças
    }
  });

  observer.observe(document.body, { childList: true, subtree: true });
}
```

### Melhoria de Lista de Conversas

```javascript
function enhanceChatList() {
  const chatListSelector = '.conversation-list';

  // Adicionar indicadores visuais para prioridade nas conversas
  const enhanceConversationItems = () => {
    document.querySelectorAll('.conversation-item').forEach(item => {
      // Verificar se já foi customizado
      if (item.dataset.customized) return;
      item.dataset.customized = 'true';

      // Acessar dados da conversa via atributos de dados
      const priority = item.getAttribute('data-priority') || 'none';

      // Aplicar estilos baseados na prioridade
      const priorityColors = {
        urgent: '#7C3AED',
        high: '#EF4444',
        medium: '#F59E0B',
        low: '#10B981',
        none: 'transparent',
      };

      // Adicionar marcador visual
      if (priority !== 'none') {
        const marker = document.createElement('div');
        marker.className = 'priority-marker';
        marker.style.width = '4px';
        marker.style.height = '100%';
        marker.style.position = 'absolute';
        marker.style.left = '0';
        marker.style.top = '0';
        marker.style.backgroundColor = priorityColors[priority];
        item.style.position = 'relative';
        item.style.paddingLeft = '4px';

        // Inserir como primeiro filho
        item.insertBefore(marker, item.firstChild);
      }
    });
  };

  // Observer para detectar novas conversas sendo adicionadas à lista
  const observer = new MutationObserver(mutations => {
    enhanceConversationItems();
  });

  // Iniciar observação quando a lista de conversas for carregada
  const waitForChatList = setInterval(() => {
    const chatList = document.querySelector(chatListSelector);
    if (chatList) {
      clearInterval(waitForChatList);
      observer.observe(chatList, { childList: true, subtree: true });
      enhanceConversationItems(); // Aplicar para itens já carregados
    }
  }, 1000);
}
```

### Atalhos e Automações

```javascript
function addKeyboardShortcuts() {
  // Registrar atalhos de teclado
  document.addEventListener('keydown', e => {
    // Alt+K para abrir o Kanban
    if (e.altKey && e.code === 'KeyK') {
      e.preventDefault();
      navigateToKanban();
    }

    // Alt+N para criar um novo item no Kanban
    if (e.altKey && e.code === 'KeyN') {
      e.preventDefault();
      createNewKanbanItem();
    }

    // Alt+F para focar na busca
    if (e.altKey && e.code === 'KeyF') {
      e.preventDefault();
      focusSearch();
    }
  });

  function navigateToKanban() {
    // Usar router do Vue para navegar
    const vueInstance = window.__vue__;
    if (vueInstance && vueInstance.$router) {
      vueInstance.$router.push({ name: 'kanban_board' });
    }
  }

  function createNewKanbanItem() {
    // Simular clique no botão de adicionar do Kanban
    const addButton = document.querySelector('.kanban-header .add-item-button');
    if (addButton) addButton.click();
  }

  function focusSearch() {
    // Focar no campo de busca
    const searchInput = document.querySelector('.search-input');
    if (searchInput) searchInput.focus();
  }

  // Adicionar painel de atalhos
  function addShortcutsPanel() {
    const panel = document.createElement('div');
    panel.className = 'shortcuts-panel';
    panel.style.position = 'fixed';
    panel.style.bottom = '20px';
    panel.style.right = '20px';
    panel.style.backgroundColor = '#ffffff';
    panel.style.padding = '8px 12px';
    panel.style.borderRadius = '8px';
    panel.style.boxShadow = '0 4px 8px rgba(0, 0, 0, 0.1)';
    panel.style.fontSize = '12px';
    panel.style.zIndex = '9999';
    panel.style.display = 'none';

    panel.innerHTML = `
      <h4 style="margin: 0 0 8px;">Atalhos</h4>
      <ul style="margin: 0; padding: 0 0 0 16px;">
        <li>Alt+K: Abrir Kanban</li>
        <li>Alt+N: Novo Item</li>
        <li>Alt+F: Busca</li>
      </ul>
    `;

    document.body.appendChild(panel);

    // Mostrar/esconder com Alt+H
    document.addEventListener('keydown', e => {
      if (e.altKey && e.code === 'KeyH') {
        e.preventDefault();
        panel.style.display = panel.style.display === 'none' ? 'block' : 'none';
      }
    });
  }

  addShortcutsPanel();
}
```

## Arquitetura Vue e Estado Global

O Chatwoot é construído com Vue.js, o que permite acessar e modificar o estado da aplicação através da instância Vue global:

```javascript
function accessVueState() {
  const vueInstance = window.__vue__;
  if (!vueInstance) return;

  // Acessar Vuex store
  const store = vueInstance.$store;

  // Exemplos de dados disponíveis no store
  const currentUser = store.getters.getCurrentUser;
  const conversations = store.getters.getConversations;
  const kanbanItems = store.getters['kanban/getKanbanItems'];

  console.log('Usuário atual:', currentUser);

  // Despachar ações para o store
  // Exemplo: Atualizar status de um item do kanban
  store.dispatch('kanban/updateItemStatus', {
    itemId: 123,
    status: 'won',
  });

  // Escutar mudanças no store
  store.subscribe((mutation, state) => {
    if (mutation.type === 'kanban/SET_KANBAN_ITEMS') {
      console.log('Itens do Kanban atualizados:', state.kanban.kanbanItems);
    }
  });
}
```

## Depuração e Desenvolvimento

```javascript
function setupDebugTools() {
  // Criar ferramenta de depuração
  const debugPanel = document.createElement('div');
  debugPanel.className = 'custom-debug-panel';
  debugPanel.style.position = 'fixed';
  debugPanel.style.bottom = '20px';
  debugPanel.style.left = '20px';
  debugPanel.style.backgroundColor = '#ffffff';
  debugPanel.style.padding = '10px';
  debugPanel.style.borderRadius = '8px';
  debugPanel.style.boxShadow = '0 4px 8px rgba(0, 0, 0, 0.1)';
  debugPanel.style.zIndex = '9999';
  debugPanel.style.maxWidth = '300px';
  debugPanel.style.maxHeight = '400px';
  debugPanel.style.overflow = 'auto';
  debugPanel.style.display = 'none';

  debugPanel.innerHTML = `
    <h4 style="margin: 0 0 8px;">Debug Tools</h4>
    <button id="debug-inspect">Inspecionar Elemento</button>
    <button id="debug-store">Imprimir Store</button>
    <button id="debug-dom">Imprimir Hierarquia DOM</button>
    <div id="debug-output" style="margin-top: 8px; font-size: 12px;"></div>
  `;

  document.body.appendChild(debugPanel);

  // Toggle com Alt+D
  document.addEventListener('keydown', e => {
    if (e.altKey && e.code === 'KeyD') {
      e.preventDefault();
      debugPanel.style.display =
        debugPanel.style.display === 'none' ? 'block' : 'none';
    }
  });

  // Implementar funcionalidades
  document.getElementById('debug-inspect').addEventListener('click', () => {
    enableElementInspector();
  });

  document.getElementById('debug-store').addEventListener('click', () => {
    printVuexStore();
  });

  document.getElementById('debug-dom').addEventListener('click', () => {
    printDOMHierarchy();
  });

  function enableElementInspector() {
    // Implementar inspetor de elementos
    alert('Clique em qualquer elemento para inspecionar');

    const onClick = e => {
      e.preventDefault();
      e.stopPropagation();

      const el = e.target;
      const output = document.getElementById('debug-output');

      output.innerHTML = `
        <strong>Element:</strong> ${el.tagName.toLowerCase()}<br>
        <strong>Classes:</strong> ${el.className}<br>
        <strong>ID:</strong> ${el.id || 'none'}<br>
        <strong>Text:</strong> ${el.textContent.substring(0, 50)}...
      `;

      document.removeEventListener('click', onClick, true);
    };

    document.addEventListener('click', onClick, true);
  }

  function printVuexStore() {
    const vueInstance = window.__vue__;
    if (!vueInstance || !vueInstance.$store) {
      alert('Vue store não encontrado');
      return;
    }

    const output = document.getElementById('debug-output');
    const store = vueInstance.$store;

    try {
      // Listar modules e getters
      const modules = Object.keys(store._modules.root._children || {});

      output.innerHTML = `
        <strong>Modules:</strong> ${modules.join(', ')}<br>
        <strong>Current User:</strong> ${store.getters.getCurrentUser?.name || 'N/A'}<br>
        <strong>Kanban Enabled:</strong> ${store.getters['kanban/isKanbanEnabled']}<br>
      `;
    } catch (e) {
      output.innerHTML = `Erro: ${e.message}`;
    }
  }

  function printDOMHierarchy() {
    const output = document.getElementById('debug-output');

    // Componentes importantes para mapear
    const components = [
      { selector: '.kanban-container', name: 'KanbanContainer' },
      { selector: '.conversation-list', name: 'ConversationList' },
      { selector: '.sidebar', name: 'Sidebar' },
      { selector: '.conversation-view', name: 'ConversationView' },
    ];

    let html = '';
    components.forEach(comp => {
      const el = document.querySelector(comp.selector);
      html += `<strong>${comp.name}:</strong> ${el ? 'Encontrado' : 'Não encontrado'}<br>`;
    });

    output.innerHTML = html;
  }
}
```

## Como Implementar

Para implementar qualquer um desses exemplos, você pode:

1. Acesse **Super Admin** > **Dashboard Scripts**
2. Cole o código desejado
3. Envolva em uma tag `<script>` se necessário
4. Combine múltiplas personalizações em um único script

### Template Completo

```javascript
<script>
document.addEventListener('DOMContentLoaded', function() {
  // Aguardar carregamento completo da aplicação Vue
  const waitForVue = setInterval(() => {
    if (window.__vue__) {
      clearInterval(waitForVue);
      initializeCustomizations();
    }
  }, 500);

  function initializeCustomizations() {
    console.log('Dashboard script inicializado');

    // Uncomment as needed:
    // customizeKanbanItems();
    // enhanceKanbanHeader();
    // customizeSidebar();
    // enhanceChatList();
    // addKeyboardShortcuts();
    // setupDebugTools();
  }

  // Paste functions here...
});
</script>
```
