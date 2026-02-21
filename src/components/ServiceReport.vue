<template>
   <div class="page">
      <div class="container">
         <header class="app-header">
            <h1>Ordem de Serviço</h1>
            <p class="subtitle">Gerador de Relatórios Profissionais</p>
         </header>

         <!-- Cliente -->
         <section class="card">
            <h2>Dados do Cliente</h2>
            <div class="grid">
               <div class="input-group">
                  <label>Nome do Cliente</label>
                  <input v-model="cliente.nome" placeholder="Ex: João Silva" />
               </div>
               <div class="input-group">
                  <label>CPF/CNPJ</label>
                  <input v-model="cliente.documento" placeholder="000.000.000-00" />
               </div>
               <div class="input-group">
                  <label>Telefone</label>
                  <input v-model="cliente.telefone" placeholder="(00) 00000-0000" />
               </div>
            </div>
         </section>

         <!-- Serviço -->
         <section class="card">
            <h2>Adicionar Serviço</h2>
            <div class="grid service-grid">
               <div class="input-group full-width">
                  <label>Descrição</label>
                  <input v-model="novoServico.nome" placeholder="Descrição do serviço realizado" />
               </div>
               <div class="input-group">
                  <label>Qtd</label>
                  <input v-model.number="novoServico.quantidade" type="number" min="1" />
               </div>
               <div class="input-group">
                  <label>Valor Unitário (R$)</label>
                  <input v-model.number="novoServico.valor" type="number" min="0" step="0.01" />
               </div>
               <div class="input-group align-end">
                  <button @click="adicionarServico" class="btn-add">Adicionar Item</button>
               </div>
            </div>
         </section>

         <!-- Produtos -->
         <section class="card">
            <h2>Adicionar Produto</h2>
            <div class="grid service-grid">
               <div class="input-group full-width">
                  <label>Descrição</label>
                  <input v-model="novoProduto.nome" placeholder="Descrição do produto" />
               </div>
               <div class="input-group">
                  <label>Qtd</label>
                  <input v-model.number="novoProduto.quantidade" type="number" min="1" />
               </div>
               <div class="input-group">
                  <label>Valor Unitário (R$)</label>
                  <input v-model.number="novoProduto.valor" type="number" min="0" step="0.01" />
               </div>
               <div class="input-group align-end">
                  <button @click="adicionarProduto" class="btn-add">Adicionar Item</button>
               </div>
            </div>
         </section>

         <!-- Lista -->
         <section v-if="servicos.length" class="card">
            <h2>Itens do Serviço</h2>
            <div class="table-responsive">
               <table class="app-table">
                  <thead>
                     <tr>
                        <th>Descrição</th>
                        <th class="text-center">Qtd</th>
                        <th class="text-right">Unitário</th>
                        <th class="text-right">Total</th>
                        <th class="text-center">Ações</th>
                     </tr>
                  </thead>
                  <tbody>
                     <tr v-for="(s, i) in servicos" :key="i">
                        <td>{{ s.nome }}</td>
                        <td class="text-center">{{ s.quantidade }}</td>
                        <td class="text-right">{{ moeda(s.valor) }}</td>
                        <td class="text-right">{{ moeda(s.quantidade * s.valor) }}</td>
                        <td class="text-center">
                           <button class="btn-remove" @click="removerServico(i)" title="Remover item">
                              <span class="icon">✕</span>
                           </button>
                        </td>
                     </tr>
                  </tbody>
               </table>
            </div>

            <div class="total-section">
               <div class="total-label">Total Geral</div>
               <div class="total-value">{{ moeda(totalServicos) }}</div>
            </div>
         </section>

         <!-- Lista -->
         <section v-if="produtos.length" class="card">
            <h2>Itens do Produto</h2>
            <div class="table-responsive">
               <table class="app-table">
                  <thead>
                     <tr>
                        <th>Descrição</th>
                        <th class="text-center">Qtd</th>
                        <th class="text-right">Unitário</th>
                        <th class="text-right">Total</th>
                        <th class="text-center">Ações</th>
                     </tr>
                  </thead>
                  <tbody>
                     <tr v-for="(p, i) in produtos" :key="i">
                        <td>{{ p.nome }}</td>
                        <td class="text-center">{{ p.quantidade }}</td>
                        <td class="text-right">{{ moeda(p.valor) }}</td>
                        <td class="text-right">{{ moeda(p.quantidade * p.valor) }}</td>
                        <td class="text-center">
                           <button class="btn-remove" @click="removerProduto(i)" title="Remover item">
                              <span class="icon">✕</span>
                           </button>
                        </td>
                     </tr>
                  </tbody>
               </table>
            </div>

            <div class="total-section">
               <div class="total-label">Total Geral</div>
               <div class="total-value">{{ moeda(totalProdutos) }}</div>
            </div>
         </section>

         <section class="card">
            <h2>Total Geral</h2>
            <div class="total-section">
               <div class="total-label">Total Geral</div>
               <div class="total-value">{{ moeda(totalGeral) }}</div>
            </div>
         </section>

         <div class="actions">
            <button class="btn-pdf" @click="gerarPDF" :disabled="gerandoPDF">
               {{ gerandoPDF ? "Gerando PDF..." : "Baixar Relatório PDF" }}
            </button>
         </div>
      </div>

      <!-- ÁREA DO PDF (Renderizada fora da tela para evitar flicker, mas visível para o html2canvas) -->
      <div class="pdf-wrapper">
         <div ref="pdfRef" class="pdf-content" :class="`fmt-${formato}`">
            <!-- Header do PDF -->
            <header class="pdf-header">
               <div class="pdf-logo-container">
                  <img :src="logo" class="pdf-logo" alt="Logo da Empresa" />
               </div>
               <div class="pdf-company-info">
                  <h1>RELATÓRIO DE SERVIÇOS</h1>
                  <div class="pdf-meta">
                     <p><strong>Nº OS:</strong> {{ numero }}</p>
                     <p><strong>Emissão:</strong> {{ dataAtual }}</p>
                  </div>
               </div>
            </header>

            <div class="pdf-divider"></div>

            <!-- Dados do Cliente -->
            <section class="pdf-section">
               <h3 class="pdf-section-title">Dados do Cliente</h3>
               <div class="pdf-client-info">
                  <div class="pdf-info-row">
                     <span class="label">Cliente:</span>
                     <span class="value">{{ cliente.nome || "N/A" }}</span>
                  </div>
                  <div class="pdf-info-row">
                     <span class="label">Documento:</span>
                     <span class="value">{{ cliente.documento || "N/A" }}</span>
                  </div>
                  <div class="pdf-info-row">
                     <span class="label">Telefone:</span>
                     <span class="value">{{ cliente.telefone || "N/A" }}</span>
                  </div>
               </div>
            </section>

            <!-- Tabela de Serviços -->
            <section class="pdf-section" v-if="servicos.length">
               <h3 class="pdf-section-title">Detalhamento dos Serviços</h3>
               <table class="pdf-table">
                  <thead>
                     <tr>
                        <th style="width: 40%">Descrição</th>
                        <th style="width: 15%" class="text-center">Qtd</th>
                        <th style="width: 20%" class="text-right">Unit.</th>
                        <th style="width: 25%" class="text-right">Total</th>
                     </tr>
                  </thead>
                  <tbody>
                     <tr v-for="(s, i) in servicos" :key="'pdf' + i">
                        <td :data-label="'Descrição'">{{ s.nome }}</td>
                        <td class="text-center" :data-label="'Qtd'">{{ s.quantidade }}</td>
                        <td class="text-right" :data-label="'Unit.'">{{ moeda(s.valor) }}</td>
                        <td class="text-right font-bold" :data-label="'Total'">{{ moeda(s.quantidade * s.valor) }}</td>
                     </tr>
                  </tbody>
                  <tfoot>
                     <tr class="pdf-total-row">
                        <td colspan="3" class="text-right label">Sub Total:</td>
                        <td class="text-right value">{{ moeda(totalServicos) }}</td>
                     </tr>
                  </tfoot>
               </table>
            </section>

            <!-- Tabela de Produtos -->
            <section class="pdf-section" v-if="produtos.length">
               <h3 class="pdf-section-title">Detalhamento dos Produtos</h3>
               <table class="pdf-table">
                  <thead>
                     <tr>
                        <th style="width: 40%">Descrição</th>
                        <th style="width: 15%" class="text-center">Qtd</th>
                        <th style="width: 20%" class="text-right">Unit.</th>
                        <th style="width: 25%" class="text-right">Total</th>
                     </tr>
                  </thead>
                  <tbody>
                     <tr v-for="(s, i) in produtos" :key="'pdf' + i">
                        <td>{{ s.nome }}</td>
                        <td class="text-center">{{ s.quantidade }}</td>
                        <td class="text-right">{{ moeda(s.valor) }}</td>
                        <td class="text-right font-bold">{{ moeda(s.quantidade * s.valor) }}</td>
                     </tr>
                  </tbody>
                  <tfoot>
                     <tr class="pdf-total-row">
                        <td colspan="3" class="text-right label">Sub Total:</td>
                        <td class="text-right value">{{ moeda(totalProdutos) }}</td>
                     </tr>
                  </tfoot>
               </table>
            </section>

            <div class="pdf-divider"></div>

            <section class="pdf-section">
               <h3 class="pdf-section-title">Total Geral</h3>
               <div class="pdf-total-geral">
                  <div class="pdf-info-row">
                     <span class="label">Total Serviços:</span>
                     <span class="value">{{ moeda(totalServicos) }}</span>
                  </div>
                  <div class="pdf-info-row">
                     <span class="label">Total Produtos:</span>
                     <span class="value">{{ moeda(totalProdutos) }}</span>
                  </div>
                  <div class="pdf-info-row">
                     <span class="label">Total Geral:</span>
                     <span class="value">{{ moeda(totalServicos + totalProdutos) }}</span>
                  </div>
               </div>
            </section>

            <!-- Observações / Footer -->
            <footer class="pdf-footer">
               <div class="pdf-footer-content">
                  <p>Este documento não possui valor fiscal.</p>
                  <p>Obrigado pela preferência!</p>
                  <div class="pdf-pagination">Página <span class="page-number"></span></div>
               </div>
               <div class="pdf-footer-brand">Gerado via Sistema de Orçamentos</div>
            </footer>
         </div>
      </div>
   </div>
</template>

<script setup lang="ts">
import { reactive, ref, computed, nextTick } from "vue";
import html2pdf from "html2pdf.js";
import logo from "@/assets/logo.png";

interface Servico {
   nome: string;
   quantidade: number;
   valor: number;
}

interface Produto {
   nome: string;
   quantidade: number;
   valor: number;
}

const cliente = reactive({
   nome: "",
   documento: "",
   telefone: "",
});

const novoServico = reactive<Servico>({
   nome: "",
   quantidade: 1,
   valor: 0,
});

const novoProduto = reactive<Produto>({
   nome: "",
   quantidade: 1,
   valor: 0,
});

const produtos = ref<Produto[]>([]);
const servicos = ref<Servico[]>([]);
const pdfRef = ref<HTMLElement | null>(null);
const gerandoPDF = ref(false);
const formato = ref<'a4' | 'letter' | 'a5' | 'mobile'>('a4');

const adicionarServico = () => {
   if (!novoServico.nome || novoServico.quantidade <= 0 || novoServico.valor < 0) return;
   servicos.value.push({ ...novoServico });
   novoServico.nome = "";
   novoServico.quantidade = 1;
   novoServico.valor = 0;
};

const adicionarProduto = () => {
   if (!novoProduto.nome || novoProduto.quantidade <= 0 || novoProduto.valor < 0) return;
   produtos.value.push({ ...novoProduto });
   novoProduto.nome = "";
   novoProduto.quantidade = 1;
   novoProduto.valor = 0;
};

const removerServico = (index: number) => {
   servicos.value.splice(index, 1);
};

const removerProduto = (index: number) => {
   produtos.value.splice(index, 1);
};

const totalServicos = computed(() => servicos.value.reduce((acc, s) => acc + s.quantidade * s.valor, 0));
const totalProdutos = computed(() => produtos.value.reduce((acc, p) => acc + p.quantidade * p.valor, 0));
const totalGeral = computed(() => servicos.value.reduce((acc, s) => acc + s.quantidade * s.valor, 0) + produtos.value.reduce((acc, p) => acc + p.quantidade * p.valor, 0));

const moeda = (valor: number) => valor.toLocaleString("pt-BR", { style: "currency", currency: "BRL" });

const numero = Math.floor(Math.random() * 10000);
const dataAtual = new Date().toLocaleDateString("pt-BR", {
   day: "2-digit",
   month: "2-digit",
   year: "numeric",
   hour: "2-digit",
   minute: "2-digit",
});

const gerarPDF = async () => {
   if (!pdfRef.value || gerandoPDF.value) return;

   gerandoPDF.value = true;

   try {
      await nextTick();

      // Espera imagens carregarem
      const images = pdfRef.value.querySelectorAll("img");
      await Promise.all(
         Array.from(images).map(
            (img) =>
               new Promise((resolve) => {
                  if (img.complete) resolve(true);
                  else {
                     img.onload = () => resolve(true);
                     img.onerror = () => resolve(true); // Resolve mesmo com erro para não travar
                  }
               }),
         ),
      );

      const formatMap: Record<string, any> = {
         a4: 'a4',
         letter: 'letter',
         a5: 'a5',
         mobile: [80, 200]
      };
      const widthMap: Record<string, number> = {
         a4: 1280,
         letter: 1280,
         a5: 992,
         mobile: 390
      };
      const options = {
         margin: [0, 0, 0, 0], // Usamos apenas o padding interno do template
         filename: `OS-${numero}-${cliente.nome.replace(/\s+/g, "_") || "cliente"}.pdf`,
         image: { type: "jpeg", quality: 0.98 },
         html2canvas: {
            scale: 2,
            useCORS: true,
            logging: false,
            scrollY: 0,
            windowWidth: widthMap[formato.value],
         },
         jsPDF: {
            unit: "mm",
            format: formatMap[formato.value],
            orientation: "portrait",
         },
         pagebreak: {
            mode: ["avoid-all", "css", "legacy"],
         },
      };

      await html2pdf().set(options).from(pdfRef.value).save();
   } catch (error) {
      console.error("Erro ao gerar PDF:", error);
      alert("Ocorreu um erro ao gerar o PDF. Tente novamente.");
   } finally {
      gerandoPDF.value = false;
   }
};
</script>

<style scoped>
/* Reset e Base */
* {
   box-sizing: border-box;
}

.page {
   width: 100%;
   min-height: 100vh;
   background: #f0f2f5;
   padding: 40px 20px;
   font-family: "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
   color: #333;
}

.container {
   max-width: 900px;
   margin: 0 auto;
}

/* Header da Aplicação */
.app-header {
   text-align: center;
   margin-bottom: 30px;
}

.app-header h1 {
   margin: 0;
   color: #1e293b;
   font-size: 2.5rem;
}

.subtitle {
   color: #64748b;
   margin-top: 5px;
}

/* Cards */
.card {
   background: white;
   padding: 25px;
   border-radius: 12px;
   box-shadow:
      0 4px 6px -1px rgba(0, 0, 0, 0.1),
      0 2px 4px -1px rgba(0, 0, 0, 0.06);
   margin-bottom: 25px;
   border: 1px solid #e2e8f0;
}

.card h2 {
   margin-top: 0;
   margin-bottom: 20px;
   color: #0f172a;
   font-size: 1.25rem;
   border-bottom: 2px solid #f1f5f9;
   padding-bottom: 10px;
}

/* Grids e Inputs */
.grid {
   display: grid;
   grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
   gap: 20px;
}

.service-grid {
   grid-template-columns: 2fr 1fr 1fr auto;
   align-items: end;
}

.input-group {
   display: flex;
   flex-direction: column;
   gap: 5px;
}

.input-group label {
   font-size: 0.875rem;
   font-weight: 500;
   color: #475569;
}

input {
   padding: 10px 12px;
   border-radius: 6px;
   border: 1px solid #cbd5e1;
   font-size: 1rem;
   transition: border-color 0.2s;
}

input:focus {
   outline: none;
   border-color: #3b82f6;
   box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

/* Botões */
button {
   padding: 10px 20px;
   border-radius: 6px;
   border: none;
   cursor: pointer;
   font-weight: 600;
   transition: all 0.2s;
   display: inline-flex;
   align-items: center;
   justify-content: center;
}

.btn-add {
   background: #3b82f6;
   color: white;
   height: 42px; /* Alinhar com inputs */
}

.btn-add:hover {
   background: #2563eb;
}

.btn-remove {
   background: #fee2e2;
   color: #ef4444;
   padding: 6px 10px;
   border-radius: 4px;
}

.btn-remove:hover {
   background: #fecaca;
}

.btn-pdf {
   background: #10b981;
   color: white;
   padding: 12px 24px;
   font-size: 1.1rem;
   width: 100%;
   max-width: 300px;
}

.btn-pdf:hover:not(:disabled) {
   background: #059669;
   transform: translateY(-1px);
}

.btn-pdf:disabled {
   background: #a7f3d0;
   cursor: not-allowed;
   transform: none;
}

.actions {
   display: flex;
   justify-content: center;
   margin-top: 30px;
}

/* Tabela da Aplicação */
.table-responsive {
   overflow-x: auto;
}

.app-table {
   width: 100%;
   border-collapse: collapse;
   margin-top: 10px;
}

.app-table th {
   background: #f8fafc;
   padding: 12px;
   text-align: left;
   font-weight: 600;
   color: #475569;
   border-bottom: 2px solid #e2e8f0;
}

.app-table td {
   padding: 12px;
   border-bottom: 1px solid #f1f5f9;
}

.text-right {
   text-align: right;
}
.text-center {
   text-align: center;
}

.total-section {
   display: flex;
   justify-content: flex-end;
   align-items: center;
   gap: 15px;
   margin-top: 20px;
   padding-top: 20px;
   border-top: 2px solid #f1f5f9;
}

.total-label {
   font-size: 1.1rem;
   color: #64748b;
}

.total-value {
   font-size: 1.5rem;
   font-weight: 700;
   color: #0f172a;
}

/* =========================================
   PDF STYLES - Design Corporativo A4
   ========================================= */

/* Wrapper para esconder da tela mas manter renderizável */
.pdf-wrapper {
   position: fixed;
   left: -9999px;
   top: 0;
   z-index: -9999;
}

.pdf-content {
   width: 210mm;
   min-height: 297mm;
   padding: 15mm 12mm; /* aumenta margem interna direita para evitar corte visual */
   background: white;
   font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
   color: #333;
   box-sizing: border-box;
   font-size: clamp(10px, 1.2vw, 13px);
   line-height: 1.4;
}

/* PDF Header */
.pdf-header {
   display: grid;
   grid-template-columns: 30mm 1fr;
   gap: 8mm;
   align-items: center;
   border-bottom: 2px solid #1e3a8a;
   padding-bottom: 6mm;
}

.pdf-logo-container {
   width: 100%;
}

.pdf-logo {
   display: block;
   width: 100%;
   max-height: 22mm;
   object-fit: contain;
}

.pdf-company-info {
   text-align: right;
   max-width: 100%;
   padding-right: 2mm; /* folga à direita para títulos longos */
}

.pdf-company-info h1 {
   color: #1e3a8a;
   font-size: clamp(16px, 2.2vw, 26px);
   text-transform: uppercase;
   letter-spacing: 1px;
   margin: 0 0 6px 0;
   overflow-wrap: anywhere;
}

.pdf-meta p {
   margin: 3px 0;
   font-size: 14px;
   color: #555;
}

.pdf-divider {
   height: 2px;
   width: 100%;
   background: #1e3a8a;
   margin: 6mm 0;
}

/* PDF Sections */
.pdf-section {
   margin-bottom: 25px;
   break-inside: avoid;
}

.pdf-section-title {
   background: #f1f5f9;
   color: #1e3a8a;
   padding: 8px 12px;
   font-size: 16px;
   border-left: 4px solid #1e3a8a;
   margin-bottom: 15px;
   text-transform: uppercase;
}

.pdf-client-info {
   display: grid;
   grid-template-columns: repeat(auto-fit, minmax(40mm, 1fr));
   gap: 6mm;
}

.pdf-info-row {
   font-size: 14px;
}

.pdf-info-row .label {
   font-weight: bold;
   color: #555;
   display: block;
   margin-bottom: 4px;
}

.pdf-info-row .value {
   font-size: 16px;
   color: #000;
}

/* PDF Table */
.pdf-table {
   width: 100%;
   border-collapse: collapse;
   font-size: 13px;
   table-layout: fixed;
}

.pdf-table th {
   background-color: #1e3a8a;
   color: white;
   padding: 0.6rem;
   text-transform: uppercase;
   font-size: 12px;
   font-weight: 600;
   overflow-wrap: anywhere;
   vertical-align: top;
}

.pdf-table td {
   padding: 0.6rem;
   border-bottom: 1px solid #e5e7eb;
   overflow-wrap: anywhere;
   word-break: break-word;
   vertical-align: top;
}

.pdf-table tr:nth-child(even) {
   background-color: #f8fafc;
}

.pdf-total-row td {
   border-top: 2px solid #1e3a8a;
   font-size: 16px;
   font-weight: bold;
   color: #1e3a8a;
   padding-top: 15px;
}

/* PDF Footer */
.pdf-footer {
   margin-top: 40px;
   border-top: 1px solid #ccc;
   padding-top: 10px;
   font-size: 12px;
   color: #777;
   display: flex;
   justify-content: space-between;
   align-items: flex-end;
   page-break-inside: avoid;
}

.pdf-footer-content p {
   margin: 2px 0;
}

.font-bold {
   font-weight: bold;
}

/* Responsividade para telas menores (Apenas UI da App, não PDF) */
@media (max-width: 768px) {
   .service-grid {
      grid-template-columns: 1fr;
   }

   .app-header h1 {
      font-size: 1.8rem;
   }

   .btn-add {
      width: 100%;
   }
}

/* Responsividade do PDF para capturas com janela menor */
@media (max-width: 600px) {
   .pdf-header {
      grid-template-columns: 1fr;
      gap: 10px;
      align-items: start;
   }
   .pdf-company-info {
      text-align: left;
      margin-top: 8px;
   }
   .pdf-table thead {
      display: none;
   }
   .pdf-table tr {
      display: block;
      padding: 10px 0;
      border-bottom: 1px solid #e5e7eb;
   }
   .pdf-table td {
      display: flex;
      justify-content: space-between;
      gap: 8px;
   }
   .pdf-table td::before {
      content: attr(data-label);
      font-weight: 600;
      color: #555;
   }
}
</style>

<style>
/* Regras globais de impressão para ajustar exatamente ao A4 */
:root {
  --a4-width: 210mm;
  --a4-height: 297mm;
  --letter-width: 216mm;
  --letter-height: 279mm;
  --a5-width: 148mm;
  --a5-height: 210mm;
}

@page {
  size: A4 portrait;
  margin: 0;
}

@media print {
  body {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
    background: white;
  }
  .pdf-content.fmt-a4 { width: var(--a4-width); min-height: var(--a4-height); padding: 12mm 12mm; margin: 0 auto; box-sizing: border-box; }
  .pdf-content.fmt-letter { width: var(--letter-width); min-height: var(--letter-height); padding: 12mm 12mm; margin: 0 auto; box-sizing: border-box; }
  .pdf-content.fmt-a5 { width: var(--a5-width); min-height: var(--a5-height); padding: 12mm; margin: 0 auto; box-sizing: border-box; }
  .pdf-content.fmt-mobile { width: 80mm; min-height: 200mm; padding: 8mm 6mm; margin: 0 auto; box-sizing: border-box; }
  .pdf-logo { max-height: 22mm; }
  .pdf-table tr,
  .pdf-section,
  .pdf-header,
  .pdf-footer {
    break-inside: avoid;
    page-break-inside: avoid;
  }
  .page-break {
    page-break-after: always;
    break-after: page;
  }
}
</style>
