# Midea Carrier — Sales Execution Engine (SEE)
### SAP Innoweeks 2026 — Integration Track

---

## Visão Geral

Este repositório centraliza a documentação técnica de integração do projeto **SEE (Sales Execution Engine)** para a Midea Carrier, desenvolvido durante a SAP Innoweeks 2026.

O SEE automatiza o processo manual de alocação de estoque e emissão de listas de remessa, substituindo extrações via transações Z, macros Excel e regras tácitas por um pipeline assistido por IA com rastreabilidade completa.

---

## Arquitetura

```
Bubble.io (UI)
    └── Xano / Supabase (Orquestração backend)
            └── Data API (Azure Functions)
                    └── Databricks (Motor de alocação)
                            └── SAP S/4HANA (Leitura + Escrita)
                            └── Data Lake Bronze / Silver / Gold
```

**Segurança:** Radware WAF → Azure APIM (mTLS + API Key) → Azure Functions (Managed Identity) → Azure Key Vault

---

## Ambiente do Cliente

| Item | Valor |
|---|---|
| Sistema | SAP S/4HANA On-Premise |
| Versão | 2023 FPS03 |
| Módulos relevantes | SD, MM, PP |

---

## APIs SAP Mapeadas

Todas verificadas e compatíveis com **S/4HANA On-Premise 2023 FPS03**.

| API | Tipo | Compatibilidade | Uso no projeto |
|---|---|---|---|
| `API_SALES_ORDER_SRV` | OData v2 | 2023 FPS03 ✅ | Leitura da carteira de pedidos (substitui extração manual) |
| `API_OUTBOUND_DELIVERY_SRV` | OData v2 | 2023 FPS03 ✅ | Leitura de remessas e estoque disponível (substitui ZREMESSA/VL10C) |
| `API_BILLING_SRV` | OData v2 | 2023 FPS03 ✅ | Leitura de faturamento (substitui export manual de NFs) |
| `BAPI_OUTB_DELIVERY_CREATE_SLS` | BAPI/RFC | Default Release ✅ | Base do RFC ABAP custom para emissão de lista |
| `BAPI_MATERIAL_AVAILABILITY` | BAPI/RFC | Default Release ✅ | Verificação ATP clássica de estoque |
| `OP_APIAVAILTOPROMISECHECK_0001` | OData v4 | Default Release ✅ | Verificação AATP avançada (backorder, alternativas) |
| `API_BILL_OF_MATERIAL_SRV` | OData v2 | 2023 FPS03 ✅ | BOM explosion para linking EVAP/COND/GRELHA (avaliar inclusão) |
| `API_BUSINESS_PARTNER_SRV` | OData v2 | 2023 FPS03 ✅ | Master data de cliente (cidade, estado, dados fiscais) |
| `DESADV` | EDI/iFlow | N/A — referência | ASN para parceiros EDI — fora do escopo imediato |

---

## Dados fora de API SAP padrão

Estes dados não têm API SAP equivalente e devem ser mantidos em **Supabase** ou ingeridos manualmente no **Data Lake**.

| Dado | Origem atual | Destino no SEE |
|---|---|---|
| **Cotas** por canal/cliente/produto | Planilha Debora / Caio Fratta | Supabase — `quota_management` |
| **Plano de Produção** (firme + planejado) | Planilha Caio Fratta | Supabase — `production_plan` / ingestão manual |
| **Rotas e Fretes** | Planilha Deivid | Supabase — `logistics_freight_table` |
| **Hierarquia comercial** de cliente (rede, gerente, coordenador) | Planilha Caio Fratta | Supabase — `customer_master` |
| **Dimensões físicas** de produto (volume m³, peso) | Simulador / ZPP044N | `API_PRODUCT_SRV` ou Data Lake Bronze |
| **Semáforo** de priorização | Excel macro | Databricks — calculado no motor |
| **FATCOM** | Campo Z customizado SAP | Tabela Z ou campo de usuário VBAK — alinhar com time SAP |
| **Importações em trânsito** (ZMM067/ZMM066) | Transação Z | Já disponível no Data Lake Bronze |
| **Preço médio USD/BRL** | Cálculo manual | Data Lake Gold — calculado pelo Databricks |

---

## Estrutura do Repositório

```
/
├── README.md              # Este arquivo
└── mock_sap_apis.json     # Mock data das APIs + mapeamento completo de campos
```

---

## Mock Data (`mock_sap_apis.json`)

O arquivo contém:

- **Dados mockados** de todas as APIs para desenvolvimento sem acesso ao SAP do cliente
- **Estruturas não-SAP** (cotas, plano de produção, fretes, master data) para uso no Supabase
- **Mapeamento completo** de todos os campos das planilhas do cliente para campos OData ou destinos alternativos
- **`field_mapping.non_sap_fields`** — lista de campos sem equivalente padrão com orientação de tratamento

---

## Dependências Críticas para Go-Live

1. **RFC ABAP custom** de emissão de lista — desenvolvimento pendente pelo time SAP
2. **Resolução do campo FATCOM** — definir se é campo de usuário VBAK ou tabela Z separada
3. **Atributos físicos de produto no SAP** — volume, peso e dimensões devem estar corretos para o Cubo Refinamento
4. **Acesso ao ambiente S/4HANA** da Midea Carrier para testes de integração real
5. **Configuração inicial dos Cubos** — regras de priorização e refinamento precisam estar definidas antes do go-live

---

## Referências

- [SAP Business Accelerator Hub](https://api.sap.com)
- Documento de Solução: `Order Management & Inventory Priorization - Data & Innovation - Documento de Solução - v 1.2.pdf`

---

## Time

| Papel | Nome |
|---|---|
| Integration Specialist | Andres Radomsky |

> SAP Innoweeks 2026
