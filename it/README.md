# Guida alla progettazione delle API HTTP

Questa guida descrive un insieme di pratiche di progettazione relative alla tecnologia HTTP+JSON, originariamente estratta dal lavoro di [Heroku Platform API](https://devcenter.heroku.com/articles/platform-api-reference).

Questa guida contiene informazioni aggiuntive a questa API e anche nuove API interne di Heroku. Noi speriamo che sia di interesse anche per i designers fuori da Heroku.

I nostri obiettivi sono ottinere consistenza, focalizzarci sulla logica di business e allo stesso tempo evitare perdita di tempo nella progettazione. Noi cerchiamo un modo buono, consistente, ben documentato per progettare le APIs, non necessariamente un unico modo di farlo.

Assiumiamo che tu sia familiare con i principi base dei rest services HTTP+JSON, d'ora in poi 'APIs' e non copriremo tutta la parte fondamentale in tal senso in questa guida.

La guida è disponibile in diversi formati a questo link [gitbook](https://www.gitbook.com/read/book/geemus/http-api-design)

Accettiamo volentieri [contributions](CONTRIBUTING.md) per questa guida.

Vai al [Sommario](it/SUMMARY.md) per controllare gli argomenti.

### Traduzioni
 * [Versione in Portoghese](https://github.com/Gutem/http-api-design/) (based on [fba98f08b5](https://github.com/interagent/http-api-design/commit/fba98f08b50acbb08b7b30c012a6d0ca795e29ee)), di [@Gutem](https://github.com/Gutem/)
 * [Versione in Spagnolo](https://github.com/jmnavarro/http-api-design) (based on [2a74f45](https://github.com/interagent/http-api-design/commit/2a74f45b9afaf6c951352f36c3a4e1b0418ed10b)), di [@jmnavarro](https://github.com/jmnavarro/)
 * [Versione in Coreano](https://github.com/yoondo/http-api-design) (based on [f38dba6](https://github.com/interagent/http-api-design/commit/f38dba6fd8e2b229ab3f09cd84a8828987188863)), di [@yoondo](https://github.com/yoondo/)
 * [Versione in cinese semplificato](https://github.com/ZhangBohan/http-api-design-ZH_CN) (based on [337c4a0](https://github.com/interagent/http-api-design/commit/337c4a05ad08f25c5e232a72638f063925f3228a)), di [@ZhangBohan](https://github.com/ZhangBohan/)
 * [Versione in Cinese tradizionale](https://github.com/kcyeu/http-api-design) (based on [232f8dc](https://github.com/interagent/http-api-design/commit/232f8dc6a941d0b25136bf64998242dae5575f66)), di [@kcyeu](https://github.com/kcyeu/)
 * [Versione Turca](https://github.com/hkulekci/http-api-design/tree/master/tr) (based on [c03842f](https://github.com/interagent/http-api-design/commit/c03842fda80261e82860f6dc7e5ccb2b5d394d51)), di [@hkulekci](https://github.com/hkulekci/)

