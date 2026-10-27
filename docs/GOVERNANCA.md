# Registro inicial de riscos e governança

Documento vivo para a Sprint 1. Não substitui avaliação jurídica nem o Relatório de Impacto à Proteção de Dados (RIPD) quando aplicável.

## Papéis de governança

| Papel | Responsabilidade | Definição no projeto |
|---|---|---|
| Controlador | Define finalidade e meios do tratamento em uma implantação real | Organização usuária; a confirmar por implantação |
| Operador | Processa dados segundo instruções do controlador | Serviço detector; a confirmar contratualmente |
| Encarregado/DPO | Canal com titulares e ANPD | Responsável da organização; fora do grupo acadêmico |
| Model Owner | Aprova versão, limiares e rollback | `PREENCHER` |
| Data Steward | Inventário, licença, qualidade e acesso aos dados | `PREENCHER` |
| Analista revisor | Rotula casos com justificativa | `PREENCHER` |
| Security Owner | Ameaças, acesso, incidentes e testes | `PREENCHER` |

## Inventário de dados e retenção proposta

| Dado | Finalidade | Armazenamento padrão | Retenção inicial | Acesso |
|---|---|---|---|---|
| Conteúdo recebido | Inferência | Memória/fila efêmera | Até concluir/repetir a requisição; alvo máximo 24 h em falha | Serviço |
| Hash/identificador opaco | Idempotência e correlação | Banco cifrado | 90 dias | Serviço e analista autorizado |
| Decisão, score, origem, versão, latência | Auditoria e métricas | Banco cifrado | 90 dias, revisão trimestral | Analista/PO conforme perfil |
| Trecho mascarado | Explicar/revisar casos | Banco cifrado, apenas quando necessário | 30 dias ou até revisão | Analista revisor |
| Rótulo e justificativa humana | Avaliação/retreino | Dataset governado | Enquanto houver finalidade e licença; revisão semestral | Dados/ML |
| Imagem/PDF original | OCR | Temporário | Exclusão após OCR, salvo política expressa | Pipeline visual |
| Log de acesso | Segurança e responsabilização | Log imutável | 180 dias, sujeito a política institucional | Segurança/auditoria |

Prazos são hipóteses de projeto, não decisões jurídicas. Em implantação, cada dado terá finalidade, hipótese legal, controlador, localidade, compartilhamento, prazo e mecanismo de descarte definidos.

## Matriz de riscos

Escala: probabilidade (P) e impacto (I) de 1 a 5; nível inerente = P × I. O risco residual será reavaliado após teste do controle.

| ID | Risco | P | I | Nível | Controles e evidência esperada | Dono |
|---|---|---:|---:|---:|---|---|
| R-01 | Dado pessoal/segredo persiste no log | 4 | 5 | 20 | allowlist de campos, teste de redaction, inspeção de logs | Security Owner |
| R-02 | Acesso indevido ao histórico | 3 | 5 | 15 | OIDC/MFA, RBAC, menor privilégio e trilha de acesso | Security Owner |
| R-03 | Fonte sem licença ou atribuição | 3 | 4 | 12 | inventário, revisão de licença e bloqueio na ingestão | Data Steward |
| R-04 | Falso negativo permite exfiltração/ação | 4 | 5 | 20 | recall por origem, menor privilégio, confirmação humana, canary | Model Owner |
| R-05 | Falso positivo interrompe negócio | 4 | 4 | 16 | hard negatives, `REVIEW`, calibração e override auditado | Model Owner |
| R-06 | Sondagem da API melhora ataques | 3 | 4 | 12 | autenticação, rate limit, razões de alto nível e alerta | Security Owner |
| R-07 | Feedback malicioso envenena retreino | 3 | 5 | 15 | dupla revisão, proveniência, dataset imutável e validação | Data Steward |
| R-08 | XSS por ataque exibido no portal | 3 | 5 | 15 | escape contextual, CSP, texto puro e testes de segurança | Front-end/Security |
| R-09 | Drift reduz qualidade silenciosamente | 3 | 5 | 15 | métricas por segmento, alerta, shadow e rollback | Model Owner |
| R-10 | OCR perde texto oculto | 4 | 4 | 16 | múltiplos pré-processamentos, confiança e `REVIEW` | CV Owner |
| R-11 | Traduções vazam entre splits | 4 | 4 | 16 | `group_id`, deduplicação e auditoria de similaridade | Data Steward |
| R-12 | Dependência/modelo arquivado fica vulnerável | 3 | 4 | 12 | SBOM, pin de versão, scan, alternativa e revisão periódica | MLOps |

## Controles de privacidade e LGPD

- minimização: API recebe somente texto e metadados necessários;
- finalidade: inferência, auditoria e melhoria são finalidades separadas;
- transparência: documentação de tratamento e canal do controlador;
- segurança: TLS, criptografia em repouso, segredo em cofre, RBAC e auditoria;
- direitos: mecanismo para localizar, corrigir ou eliminar registros vinculáveis quando aplicável;
- transferência: verificar região e subprocessadores antes de usar nuvem/geração externa;
- dados sensíveis e de crianças: não usar no dataset; aplicar avaliação específica se surgirem em produção;
- incidente: identificar, conter, preservar evidência mínima, avaliar impacto e seguir o plano do controlador.

Conteúdo produtivo não deve ser enviado a provedores de IA generativa para gerar exemplos ou explicações. Dados sintéticos partem de cenários fictícios.

## Governança de dataset e modelo

Cada dataset terá versão, datasheet, hash, composição, fontes/licenças, splits, revisão e limitações. Cada modelo terá model card com base, hiperparâmetros/LoRA, dataset exato, métricas globais e segmentadas, limiares, artefato/hash e aprovador.

Promoção proposta:

1. execução reproduzível nos testes congelados;
2. piso de recall e teto de falso positivo definidos com o PO;
3. comparação contra versão campeã e baselines;
4. revisão de licença, privacidade e dependências;
5. teste em shadow/canary;
6. aprovação do Model Owner e plano de rollback.

Rótulos de produção nunca entram automaticamente no treino. Eles passam por mascaramento, licença/finalidade, revisão e versionamento.

## Uso indevido e divulgação

O serviço é defensivo. Acesso anônimo, enumeração ilimitada de score e publicação de regras completas podem ajudar evasão. A API exige autenticação, rate limit e razões de alto nível. Exemplos públicos devem ser suficientes para ensino e avaliação, sem dados reais, credenciais ou instruções operacionais contra sistemas existentes.

## Checklist antes de prototipar

- [ ] Definir controlador, operador, finalidade e hipótese legal.
- [ ] Aprovar retenção e procedimento de exclusão verificável.
- [ ] Completar inventário/licenças das fontes.
- [ ] Definir perfis de acesso e matriz de segregação.
- [ ] Escrever playbook de incidente e rollback de modelo.
- [ ] Definir pisos de qualidade e orçamento máximo de `REVIEW`.
- [ ] Testar redaction de logs, XSS, payloads grandes e abuso de OCR.
- [ ] Produzir datasheet, model card e relatório de avaliação.

Referência legal primária: [Lei nº 13.709/2018 — LGPD](https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm).

