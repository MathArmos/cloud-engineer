# cenarios/ — aplicações práticas dos conceitos ACE

Esta pasta é **separada de `modulos/`** de propósito.

`modulos/` segue os domínios do exame e o conteúdo dos cursos (cada arquivo amarrado a uma fonte específica — workbook ACE, Skills Boost, etc.).

`cenarios/` é onde **destilamos a teoria em decisões de arquitetura** para casos reais — geralmente conectados ao trabalho do dia a dia (Portaria, TaxIA, agent WhatsApp). Cada arquivo:

- Descreve um sistema real ou plausível
- Mapeia cada componente para o serviço GCP certo
- Explica **por que** essa escolha (e por que as alternativas falham)
- Inclui evolução por escala (tier 1 → tier 2 → tier 3)
- Termina com perguntas de feira + respostas prontas

**Propósito final:** chegar à feira de trabalho com **2 a 3 cenários defensáveis em voz alta**, conectando o vocabulário GCP à experiência prática.

---

## Arquivos

- [agent-whatsapp-regras-rigorosas.md](agent-whatsapp-regras-rigorosas.md) — arquitetura de storage/database para um agent WhatsApp que segue regras de negócio rigorosas (mapeia system prompts, knowledge base, histórico, mídia, cache e traces para serviços GCP)
