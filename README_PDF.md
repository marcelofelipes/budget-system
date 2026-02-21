# Documentação Técnica: Geração de Relatórios PDF

Este documento descreve as implementações e correções realizadas no módulo de geração de PDF (`ServiceReport.vue`).

## 1. Problema Identificado: PDF em Branco
A causa raiz do PDF em branco geralmente está associada a três fatores no uso da biblioteca `html2pdf.js` / `html2canvas`:
1.  **Visibilidade do Elemento**: O elemento a ser renderizado estava com `opacity: 0` ou `display: none`. O `html2canvas` ignora elementos que não são visíveis no DOM.
2.  **Scroll Position**: Se o usuário rolar a página para baixo, o `html2canvas` pode capturar a viewport atual e cortar o conteúdo que está no topo (`top: 0`), resultando em uma imagem branca ou cortada.
3.  **Carregamento de Imagens**: O PDF era gerado antes das imagens (logo) estarem totalmente carregadas.

## 2. Soluções Implementadas

### Correção de Renderização
- **Posicionamento Off-Screen**: Ao invés de usar `opacity: 0`, movemos o container do PDF para fora da área visível usando `position: fixed; left: -9999px; top: 0;`. Isso mantém o elemento "visível" para o navegador (renderizável) mas invisível para o usuário, garantindo que o `html2canvas` consiga capturá-lo.
- **Reset de Scroll (`scrollY: 0`)**: Configuramos a opção `scrollY: 0` no `html2canvas` para garantir que a captura comece sempre do topo do elemento, independentemente da posição de rolagem do usuário.
- **Window Width Fix**: Definimos `windowWidth: 1200` para simular uma janela de desktop durante a renderização, garantindo que o layout responsivo não "quebre" para mobile durante a geração do PDF.

### Melhorias de Layout e Design
- **Estilo Corporativo**: Aplicação de uma paleta de cores profissional (Azul Navy `#1e3a8a`) e tipografia limpa (`Helvetica Neue`).
- **Estrutura de Tabela**: Uso de tabelas com cabeçalhos claros, zebrado (striped rows) e alinhamento adequado de valores monetários.
- **Cabeçalho e Rodapé**: Inclusão de logo, dados da empresa, numeração de OS dinâmica e rodapé com paginação e termos de isenção.

### Responsividade e Adaptação
- **Layout Flexível**: O CSS do PDF utiliza porcentagens para larguras de colunas, permitindo que o conteúdo se adapte se o formato do papel mudar (ex: A4 para Letter).
- **Interface de Entrada**: A tela de preenchimento dos dados foi tornada responsiva para dispositivos móveis, separando a lógica de visualização (App) da lógica de impressão (PDF).

## 3. Checklist de Validação e Manutenção

Para futuras alterações no relatório, utilize este checklist para garantir a integridade da geração do PDF:

- [ ] **Visibilidade**: O elemento `#pdf-content` (ou ref equivalente) deve estar presente no DOM (não usar `v-if` que o remova antes da geração).
- [ ] **Imagens**: Todas as imagens locais ou remotas (`<img src="...">`) devem estar carregadas antes de chamar `.save()`. (O código atual já possui um `Promise.all` para isso).
- [ ] **CORS**: Se usar imagens externas (S3, CDN), certifique-se de que o servidor de imagens retorna headers CORS (`Access-Control-Allow-Origin: *`) e que a opção `useCORS: true` está ativa.
- [ ] **Quebras de Página**:
    - Evite elementos com `height: 100vh` dentro do PDF.
    - Use a classe CSS `page-break-inside: avoid` em containers que não devem ser cortados ao meio (ex: linhas de tabela, assinaturas).
- [ ] **Estilos Globais**: Verifique se o CSS global (`base.css` ou similar) não está interferindo (ex: Dark Mode). O container do PDF deve ter `background: white` e `color: black` explícitos.
- [ ] **Fontes**: Utilize fontes padrão (Arial, Helvetica) ou garanta que webfonts estejam carregadas antes da geração.

## 4. Como Testar
1.  Preencha os dados na interface.
2.  Adicione múltiplos itens para testar a paginação (adicione 20+ itens para forçar uma segunda página).
3.  Clique em "Baixar Relatório PDF".
4.  Verifique se:
    - O logo aparece.
    - Os acentos estão corretos (UTF-8).
    - A tabela não está cortada abruptamente.
    - O total bate com a soma dos itens.

## 5. Template A4 (Ajuste Fino)

Para deixar o relatório exatamente no formato A4:

- CSS do container do PDF:
  - Largura e altura mínimas em milímetros:
    - `.pdf-content { width: 210mm; min-height: 297mm; padding: 15mm; box-sizing: border-box; }`
  - Evite utilizar `vh`/`vw` dentro do PDF.
- Regras globais de impressão:
  - Incluímos um bloco `<style>` não escopado com:
    - `@page { size: A4 portrait; margin: 0; }`
    - `@media print { .pdf-content { width: 210mm; min-height: 297mm; padding: 15mm; } }`
- Opções do `html2pdf.js`:
  - `jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }`
  - `margin: [10, 10, 15, 10]` para margens externas do PDF
  - `html2canvas: { scale: 2, scrollY: 0, useCORS: true }`
- Quebras de página:
  - Use a classe `.page-break` onde for necessário forçar nova página.
  - Regra `break-inside: avoid` aplicada em seções, cabeçalhos e linhas de total para evitar cortes.

Para trocar para Letter:
- Ajuste `jsPDF.format` para `"letter"`.
- Opcionalmente, ajuste o CSS:
  - `--letter-width: 216mm; --letter-height: 279mm;`
  - `.pdf-content { width: 216mm; min-height: 279mm; }`

## 6. Responsividade do Documento

Objetivo: garantir que o HTML base do PDF se adapte a diferentes larguras de janela (usadas pelo `html2canvas`) e a diferentes formatos de página.

Mudanças aplicadas:
- Container `.pdf-content` com `font-size: clamp(10px, 1.2vw, 13px)`, tabelas com `table-layout: fixed`, `word-break` e `overflow-wrap`.
- Logo e imagens com `max-width: 100%` e limites por `clamp`.
- Layout do header com `flex-wrap` e reflow em janelas menores.
- Tabela responsiva:
  - Esconde `<thead>` abaixo de 600px.
  - Cada `<td>` exibe um rótulo via `content: attr(data-label)` em telas estreitas.
- Suporte a formatos:
  - Classe dinâmica em `.pdf-content`: `fmt-a4`, `fmt-letter`, `fmt-a5`, `fmt-mobile`.
  - Regras de impressão definidas por `@media print` para cada formato.
  - Mapeamento de janela virtual via `html2canvas.windowWidth` por formato.

Como testar:
1. Emular janelas com larguras 320, 768, 1024 e 1920.
2. Gerar PDF com formato `a4` e `mobile` para comparar o layout.
3. Validar que:
   - Colunas não estouram as margens.
   - Longas descrições quebram linhas corretamente.
   - Logo redimensiona proporcionalmente.
   - Rodapé permanece visível e não sobreposto.

Aplicar em outro projeto:
1. Defina um container do documento com largura/altura via mm e `box-sizing: border-box`.
2. Use `table-layout: fixed` e `data-label` nos `<td>` para responsividade.
3. Aplique `@page` e regras de `@media print` mapeadas para cada formato desejado.
4. Configure `html2canvas` com `scrollY: 0`, `scale >= 2` e `windowWidth` conforme o breakpoint alvo.
