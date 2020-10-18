---
title: Perguntas sobre JavaScript
---

Respostas para [perguntas de entrevistas Front-end - Perguntas sobre JS](https://github.com/h5bp/Front-end-Developer-Interview-Questions/blob/master/src/questions/javascript-questions.md). _Pull requests_ para sugestões e correções são bem-vindos!

## Tabela de conteúdos

- [Explique delegação de evento](#explain-event-delegation)

### Explique delegação de evento

Delegação de evento é uma técnica que envolve adicionar _listeners_ de eventos a um elemento pai ao invés de adicioná-los a elementos filhos. O _listener_ irá "escutar" sempre que o evento é disparado nos elementos filhos devido ao _bubbling_ de eventos do DOM. Os benefícios desta técnica são:

- A _Memory Footprint_ diminui porque somente um manipulador (_handler_) é necessário no elemento pai, ao invés de ter que anexar um manipulador em cada filho.
- Não há necessidade de fazer o _unbind_ do manipulador de elementos que são removidos e o _bind_ de novos elementos.

###### Referências (Inglês)

- https://davidwalsh.name/event-delegate
- https://stackoverflow.com/questions/1687296/what-is-dom-event-delegatio

##### Referências (pt-BR)

- https://developer.mozilla.org/pt-BR/docs/Aprender/JavaScript/Elementos_construtivos/Events#Delega%C3%A7%C3%A3o_de_eventos

[[↑] Voltar ao topo](#table-of-contents)
