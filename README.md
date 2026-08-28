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
                            └── Data Lake Bronze/Silver/Gold
```

**Segurança:** Radware WAF → Azure APIM → Azure Functions (Managed Identity) → Azure Key Vault

---

## Ambiente do Cliente

| Item | Valor |
|---|---|
| Sistema | SAP S/4HANA On-Premise |
| Versão | 2023 FPS03 |
| Módulos relevantes | SD (Sales & Distribution), MM, PP |

---

## APIs SAP Mapeadas

| API | Tipo | Compatibilidade | Uso no projeto |
|---|---|---|---|
| `API_SALES_ORDER_SRV` | OData v2 | 2023 FPS03 ✅ | Leitura da carteira de pedidos |
| `API_OUTBOUND_DELIVERY_SRV` | OData v2 | 2023 FPS03 ✅ | Leitura e criação de remessas |
| `BAPI_OUTB_DELIVERY_CREATE_SLS` | BAPI/RFC | Default Release ✅ | Base do RFC ABAP custom de emissão de lista |
| `BAPI_MATERIAL_AVAILABILITY` | BAPI/RFC | Default Release ✅ | Verificação ATP clássica de estoque |
| `OP_APIAVAILTOPROMISECHECK_0001` | OData v4 | Default Release ✅ | Verificação AATP avançada (backorder, alternativas) |
| `DESADV` | EDI/iFlow | N/A | Referência para ASN — fora do escopo imediato |

---

## Estrutura do Repositório

```
/
├── README.md                  # Este arquivo
├── mock_sap_apis.json         # Mock data das APIs + mapeamento de campos
└── docs/
    └── (documentos de solução e referência do cliente)
```

---

## Mock Data

O arquivo `mock_sap_apis.json` contém:

- **Dados mockados** de todas as APIs para uso no desenvolvimento sem acesso ao SAP do cliente
- **Mapeamento completo** dos campos extraídos hoje (planilhas/transações Z) para os campos OData correspondentes
- **Campos sem equivalente padrão** com orientação de tratamento

### Campos sem mapeamento direto

| Campo | Situação |
|---|---|
| `FATCOM` | Campo Z customizado — requer alinhamento com time SAP da Midea |
| `Semáforo` | Lógica de negócio — implementar no motor Databricks |
| `Cotas` | Sem API padrão — manter em Supabase/tabela de configuração |
| `Plano de Produção` | Sem API padrão — ingestão manual ou integração PP futura |
| `Nível explosão (BOM)` | Coberto por `API_BILL_OF_MATERIAL_SRV` — avaliar inclusão no escopo |

---

## Dependências Críticas para Go-Live

1. **RFC ABAP custom** de emissão de lista — desenvolvimento pendente pelo time SAP
2. **Atributos de produto no SAP** — volume, peso, dimensões devem estar corretos para o Cubo Refinamento
3. **Resolução do campo FATCOM** — definir se é exposto via extensão SD ou tabela Z separada
4. **Acesso ao ambiente S/4HANA** da Midea Carrier para testes de integração real

---

## Referências

- [SAP Business Accelerator Hub](https://api.sap.com)
- [SAP Help — API_SALES_ORDER_SRV](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE)
- Documento de Solução: `Order Management & Inventory Priorization - Data & Innovation - Documento de Solução - v 1.2.pdf`

---

## Time

| papel | Nome |
|---|---|
| Integration Specialist | Andres Radomsky |

> SAP Innoweeks 2026
