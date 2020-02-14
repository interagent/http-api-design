# Guida alla progettazione di API HTTP

Questa guida descrive un insieme di pratiche per la progettazione di API HTTP+JSON, originariamente tratte dal lavoro sulle [API della Piattaforma Heroku](https://devcenter.heroku.com/articles/platform-api-reference).

La guida serve come riferimento sia per le modifiche alle API correnti che per lo sviluppo di nuove API all'interno di Heroku; speriamo sia di interesse per tutti gli API designer anche al di fuori di Heroku.

I nostri obiettivi sono la coerenza ed il focalizzarsi sulla logica di business, evitando inutili perdite di tempo durante la progettazione. Perseguiamo un _metodo ottimale, coerente e ben documentato_ per progettare API, che non sia necessariamente _l'unico e solo modo ideale_ di farlo.

Diamo per scontato che tu abbia già familiarità con i principi alla base delle API HTTP+JSON, quindi questa guida non comprende tutta la parte teorica fondamentale sull'argomento.

La guida è disponibile in diversi formati a questo link [gitbook](https://www.gitbook.com/read/book/geemus/http-api-design)

Accettiamo volentieri [contributi](../CONTRIBUTING.md) alla guida.

Vai al [Sommario](SUMMARY.md) per l'indice dei contenuti.

### Traduzioni
 * [Versione in Portoghese](https://github.com/Gutem/http-api-design/) (based on [fba98f08b5](https://github.com/interagent/http-api-design/commit/fba98f08b50acbb08b7b30c012a6d0ca795e29ee)), di [@Gutem](https://github.com/Gutem/)
 * [Versione in Spagnolo](https://github.com/jmnavarro/http-api-design) (based on [2a74f45](https://github.com/interagent/http-api-design/commit/2a74f45b9afaf6c951352f36c3a4e1b0418ed10b)), di [@jmnavarro](https://github.com/jmnavarro/)
 * [Versione in Coreano](https://github.com/yoondo/http-api-design) (based on [f38dba6](https://github.com/interagent/http-api-design/commit/f38dba6fd8e2b229ab3f09cd84a8828987188863)), di [@yoondo](https://github.com/yoondo/)
 * [Versione in Cinese Semplificato](https://github.com/ZhangBohan/http-api-design-ZH_CN) (based on [337c4a0](https://github.com/interagent/http-api-design/commit/337c4a05ad08f25c5e232a72638f063925f3228a)), di [@ZhangBohan](https://github.com/ZhangBohan/)
 * [Versione in Cinese Tradizionale](https://github.com/kcyeu/http-api-design) (based on [232f8dc](https://github.com/interagent/http-api-design/commit/232f8dc6a941d0b25136bf64998242dae5575f66)), di [@kcyeu](https://github.com/kcyeu/)
 * [Versione in Turco](https://github.com/hkulekci/http-api-design/tree/master/tr) (based on [c03842f](https://github.com/interagent/http-api-design/commit/c03842fda80261e82860f6dc7e5ccb2b5d394d51)), di [@hkulekci](https://github.com/hkulekci/)

