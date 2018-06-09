| component | all ports | ports needed by external workers (e.g. verdi workers on cloud) |
| --- | --- | --- |
| mozart-rabbit | 5672, 15672 | 5672 |
| mozart-figaro-redis | 80, 443, 5555, 6379, 8888, 8898, 9001 | 80, 443, 6379, 8888 |
| mozart-elasticsearch | 9200, 9300 | 9200 |
| metrics | 80, 443, 6379, 9001, 9200, 9300 | 80, 443 |
| grq | 80, 443, 6379, 8878, 8879, 9001, 9200, 9300 | 80, 443 8878, 9200 |
| factotum | 25, 80, 443, 6379, 8085, 9001, 46224 | 80, 443 6379 |
| ci | 80, 443, 8080, 9001 | |