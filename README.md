# Scripts para Chatwoot

Scripts prontos para uso em sua instalação do Chatwoot que adicionam funcionalidades extras para o Kanban.

## Requisitos

- [Kanban para Chatwoot](https://kanbanparachatwoot.com.br/) - Solução completa para visualização em Kanban no Chatwoot

## Como Instalar

### Método Correto de Instalação

1. Adquira o Kanban para Chatwoot em [kanbanparachatwoot.com.br](https://kanbanparachatwoot.com.br/)
2. Acesse sua instalação do Chatwoot com uma conta de super admin
3. Navegue até `https://seuchatwoot.com/super_admin/dashboard_scripts`
4. Cole o conteúdo do script no campo "Script Code"
5. Clique em "Salvar Scripts"

## Criando Scripts Personalizados

Para criar seus próprios scripts para o Kanban para Chatwoot, siga este modelo:

```html
<script>
  document.addEventListener('DOMContentLoaded', function () {
    // Função principal para implementar a funcionalidade
    function aplicarFuncionalidade() {
      // Verificar se o elemento já existe para evitar duplicação
      if (document.getElementById('seu-elemento-id')) return;

      // 1. Criar elemento de estilo para injetar CSS personalizado
      const style = document.createElement('style');

      // 2. Definir estilos CSS
      style.innerHTML = `
        /* Seus estilos CSS aqui */
        #seu-elemento-id {
          position: fixed;
          bottom: 20px;
          right: 20px;
          /* Mais estilos */
        }
      `;

      // 3. Adicionar o estilo ao head do documento
      document.head.appendChild(style);

      // 4. Criar elementos DOM necessários
      const seuElemento = document.createElement('div');
      seuElemento.id = 'seu-elemento-id';
      seuElemento.innerHTML = `
        <!-- Seu HTML aqui -->
      `;

      // 5. Adicionar elementos ao DOM
      document.body.appendChild(seuElemento);

      // 6. Adicionar eventos e comportamentos
      seuElemento.addEventListener('click', function () {
        // Lógica do evento
      });
    }

    // Executar imediatamente
    aplicarFuncionalidade();

    // Também executar quando houver mudanças dinâmicas na DOM
    const observer = new MutationObserver(function () {
      aplicarFuncionalidade();
    });

    observer.observe(document.body, { childList: true, subtree: true });
  });
</script>
```

### Diretrizes para Criação de Scripts:

1. **Verifique duplicações**: Sempre verifique se o elemento já existe antes de adicioná-lo
2. **Use IDs únicos**: Evite conflitos com outros elementos da página
3. **Inclua um observer**: O MutationObserver garante que seu script funcione mesmo quando o DOM é alterado dinamicamente
4. **Encapsule a lógica**: Mantenha todo o código dentro da função principal para evitar poluir o namespace global
5. **Comente seu código**: Adicione comentários explicativos para facilitar a manutenção

### Exemplos de Funcionalidades:

- Botões flutuantes para ações rápidas
- Atalhos de teclado personalizados
- Integrações com APIs externas
- Personalização visual do Kanban
- Automações para processos repetitivos

### Exemplo Prático: Botão de Chamada para Kanban

Abaixo um exemplo específico de um script que adiciona um botão de chamada telefônica no Kanban:

```html
<script>
  document.addEventListener('DOMContentLoaded', function () {
    // Função principal para adicionar botão de chamada
    function adicionarBotaoChamada() {
      // Verificar se o botão já existe para evitar duplicação
      if (document.getElementById('kanban-call-button')) return;

      // 1. Criar elemento de estilo para o botão
      const style = document.createElement('style');
      style.innerHTML = `
        #kanban-call-button {
          position: fixed;
          bottom: 20px;
          right: 20px;
          width: 56px;
          height: 56px;
          border-radius: 50%;
          background-color: #84CC16;
          color: white;
          display: flex;
          align-items: center;
          justify-content: center;
          cursor: pointer;
          box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
          z-index: 9999;
          transition: all 0.3s ease;
        }
        
        #kanban-call-button:hover {
          transform: scale(1.1);
          box-shadow: 0 6px 15px rgba(0, 0, 0, 0.25);
        }
      `;

      document.head.appendChild(style);

      // 2. Criar o botão com ícone de telefone
      const button = document.createElement('div');
      button.id = 'kanban-call-button';
      button.innerHTML = `
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/>
        </svg>
      `;

      // 3. Adicionar evento de clique que extrai o telefone do card selecionado
      button.addEventListener('click', function () {
        // Obter o card selecionado no Kanban
        const selectedCard = document.querySelector('.kanban-card.selected');

        if (selectedCard) {
          // Extrair número de telefone do card (exemplo)
          const phoneElement = selectedCard.querySelector('.contact-phone');
          const phone = phoneElement ? phoneElement.textContent.trim() : '';

          if (phone) {
            // Iniciar chamada (isso depende da sua implementação)
            alert(`Iniciando chamada para: ${phone}`);
            // Aqui você poderia integrar com alguma API de chamadas
          } else {
            alert('Nenhum telefone encontrado no card selecionado');
          }
        } else {
          alert('Selecione um card do Kanban para fazer uma chamada');
        }
      });

      // 4. Adicionar o botão ao DOM
      document.body.appendChild(button);
    }

    // Executar imediatamente
    adicionarBotaoChamada();

    // Também executar quando houver mudanças dinâmicas na DOM
    const observer = new MutationObserver(function () {
      adicionarBotaoChamada();
    });

    observer.observe(document.body, { childList: true, subtree: true });
  });
</script>
```

Este exemplo demonstra como adicionar um botão flutuante que pode interagir com os cards do Kanban. Você pode personalizar a funcionalidade conforme suas necessidades específicas.

## Contribuições

Se você possui sugestões para melhorar os scripts e a integração, entre em contato através da comunidade no Whatsapp do Kanban para Chatwoot.

## Suporte

Para suporte e dúvidas sobre a instalação, acesse a comunidade no Whatsapp do Kanban para Chatwoot após a compra.

