# F1-E07 — Relatórios administrativos e oficiais

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E07 — Relatórios administrativos e oficiais |
| Fase | F1 (MVP) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | `reporting` |
| Depende de | F1-E01 a F1-E06 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E07` para histórias/critérios completos (referência, não leitura obrigatória) |

## Objetivo

Gerar relatórios operacionais filtráveis (listas de alunos, frequência, notas, resultados finais, inadimplência), exportáveis em PDF/planilha, e exportar dados no formato exigido pelo Censo Escolar/INEP.

## Histórias cobertas

- `F1-E07-U01` — Relatórios filtráveis por turma/série/período/situação, exportáveis em PDF/planilha.
- `F1-E07-U02` — Exportação de dados no formato do Censo Escolar/INEP.

## Multi-tenancy

Mecanismo completo em [`arquitetura-ignite.md` §11](../../../arquitetura-ignite.md#11-multi-tenancy-fundacional-desde-o-dia-zero) e [`F1-E0A`](../F1-E0A-tenancy/spec-tenancy.md). Específico deste épico:

- Todo `Generate*ReportUseCase` já herda o filtro de `groupId`/`allowedSchoolIds` da Prisma extension através dos repositórios injetados — nenhum filtro manual de tenant precisa ser escrito nos use cases de relatório em si, só os filtros de negócio (`turmaId`, `serieId`, `período`).
- **Censo Escolar/INEP é por escola**, não por group — cada escola tem seu próprio código INEP (`School.inepCode`, de `F1-E0A`). `ExportCensoEscolarUseCase` recebe `schoolId` (não só `schoolYearId`) e gera um arquivo por escola; se o usuário tiver acesso a várias escolas, a tela do frontend oferece exportar uma de cada vez ou em lote (decisão de UX, não de use case).
- `GenerateFinalResultsReportUseCase`/demais relatórios filtráveis por "turma/série" só listam turmas/séries dentro do escopo de escolas do usuário — resultado natural do filtro de `EnrollmentsRepository`/`TurmasRepository`, sem lógica extra.

## Modelo de domínio (`enterprise`)

**N/A.** `reporting` é uma camada de leitura pura sobre os agregados já existentes em `people`, `academic`, `attendance`, `assessment` e `finance` — não introduz entidades/VOs novos. Os use cases desta ficha injetam diretamente os repositórios já existentes desses contextos (leitura cross-context é aceitável aqui porque `reporting` não escreve em nenhum deles).

## Use cases (`application`)

### `conaa-api/src/domain/reporting/application/useCases/`

| Use case | Arquivo | Entrada | Repositórios injetados | Saída (`Either`) |
| --- | --- | --- | --- | --- |
| `GenerateStudentListReportUseCase` | `generate-student-list-report.ts` | `turmaId?`, `serieId?`, `status?` | `StudentsRepository`, `EnrollmentsRepository` | `Either<never, { rows: StudentReportRow[] }>` |
| `GenerateAttendanceReportUseCase` | `generate-attendance-report.ts` | `turmaId?`, `serieId?`, `period?` | `AttendanceRecordsRepository` | `Either<never, { rows: AttendanceReportRow[] }>` |
| `GenerateGradesReportUseCase` | `generate-grades-report.ts` | `turmaId?`, `serieId?`, `period?` | `GradesRepository`, `AssessmentsRepository` | `Either<never, { rows: GradesReportRow[] }>` |
| `GenerateFinalResultsReportUseCase` | `generate-final-results-report.ts` | `turmaId?`, `serieId?`, `schoolYearId` | `EnrollmentsRepository` | `Either<never, { rows: FinalResultReportRow[] }>` |
| `GenerateDelinquencyReportUseCase` | `generate-delinquency-report.ts` | `turmaId?`, `serieId?`, `period?` | `InvoicesRepository` (mesma agregação de `GetFinancialStatusUseCase` de `F1-E06`, reaproveitar via injeção do repositório, não duplicar lógica) | `Either<never, { rows: DelinquencyReportRow[] }>` |
| `ExportCensoEscolarUseCase` | `export-censo-escolar.ts` | `schoolId`, `schoolYearId` | `StudentsRepository`, `EnrollmentsRepository` | `Either<IncompleteCensusDataError, { records: CensoRecord[] }>` |

`*ReportRow`/`CensoRecord` são tipos simples (DTOs) declarados junto de cada use case, não entidades de domínio.

Regras de negócio principais:
- Todos os relatórios de listagem (`Generate*ReportUseCase`) retornam `Either<never, ...>` — filtro vazio é um resultado válido (lista vazia), não um erro.
- `ExportCensoEscolarUseCase`: para cada aluno da lista, valida campos obrigatórios do Censo (CPF, data de nascimento, endereço completo — o conjunto exato de campos obrigatórios do layout vigente deve ser confirmado contra a especificação oficial do INEP do ano corrente antes de codar; até lá, implementar com um `TODO` explícito no código listando os campos assumidos). Retorna `IncompleteCensusDataError` listando quais alunos têm pendência, sem impedir a geração de um relatório parcial (a exportação em si é bloqueada só se _nenhum_ aluno estiver completo).

## Persistência (`infra/database/prisma`)

Nenhum novo model. Os repositórios usados já existem (`F1-E01`, `F1-E02`, `F1-E03`, `F1-E04`, `F1-E06`); esta ficha só adiciona, quando necessário, métodos de leitura agregada nesses repositórios (ex.: `EnrollmentsRepository.findManyBySerieWithResult`), sempre no arquivo do repositório já existente — **não** criar um novo repositório "de relatórios" que duplica queries.

## HTTP (`infra/http`)

| Rota | Controller | Query params | Formato de saída |
| --- | --- | --- | --- |
| `GET /reports/students` | `generate-student-list-report.controller.ts` | `turmaId?, serieId?, status?, format=pdf\|xlsx` | arquivo (PDF ou planilha) |
| `GET /reports/attendance` | `generate-attendance-report.controller.ts` | `turmaId?, serieId?, period?, format` | arquivo |
| `GET /reports/grades` | `generate-grades-report.controller.ts` | `turmaId?, serieId?, period?, format` | arquivo |
| `GET /reports/final-results` | `generate-final-results-report.controller.ts` | `turmaId?, serieId?, schoolYearId, format` | arquivo |
| `GET /reports/delinquency` | `generate-delinquency-report.controller.ts` | `turmaId?, serieId?, period?, format` | arquivo |
| `GET /reports/censo-escolar` | `export-censo-escolar.controller.ts` | `schoolId, schoolYearId, format=csv` | arquivo CSV |

Geração de arquivo: PDF reaproveita a mesma lib decidida em `F1-E04` (`pdf-lib` ou equivalente definido no bootstrap); planilha usa `exceljs` (XLSX) ou `csv-stringify` (CSV) — preferir `csv-stringify` para o Censo por ser o formato mais comum de importação em sistemas do MEC, e `exceljs` só para os relatórios operacionais que pedem XLSX explicitamente.

Perfis exigidos: todas as rotas → `direção`/`coordenação`/`secretaria` (guard fino vem de `F1-E09`; por ora `JwtAuthGuard` padrão).

## Frontend (`conaa-web`)

- `app/(portal)/relatorios/page.tsx` — hub de relatórios com seleção de tipo + filtros (turma/série/período/situação) + botões de exportação (`F1-E07-U01`).
- `app/(portal)/relatorios/censo-escolar/page.tsx` — tela dedicada de exportação do Censo, com lista de alunos pendentes destacada antes de exportar (`F1-E07-U02`).
- `features/reporting/components/ReportFilters.tsx`, `ReportTable.tsx`, `CensoPendencyList.tsx`.
- `features/reporting/api/reports.ts` (`generateStudentListReport`, `generateAttendanceReport`, `generateGradesReport`, `generateFinalResultsReport`, `generateDelinquencyReport`, `exportCensoEscolar` — todas retornam blob para download), `features/reporting/schemas/report-filters.ts`.

## Arquivos a criar/editar (checklist)

- [ ] `conaa-api/src/domain/reporting/application/useCases/generate-student-list-report.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/reporting/application/useCases/generate-attendance-report.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/reporting/application/useCases/generate-grades-report.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/reporting/application/useCases/generate-final-results-report.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/reporting/application/useCases/generate-delinquency-report.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/reporting/application/useCases/export-censo-escolar.ts` (+ errors/, + .spec.ts)
- [ ] `conaa-api/src/infra/http/controllers/*.ts` (6 controllers, ver seção HTTP)
- [ ] `conaa-api/src/infra/http/presenters/report-file-presenter.ts` (helper compartilhado: monta `Content-Disposition`/`Content-Type` conforme `format`)
- [ ] `conaa-web/app/(portal)/relatorios/page.tsx`
- [ ] `conaa-web/app/(portal)/relatorios/censo-escolar/page.tsx`
- [ ] `conaa-web/features/reporting/components/*.tsx`
- [ ] `conaa-web/features/reporting/api/reports.ts`
- [ ] `conaa-web/features/reporting/schemas/report-filters.ts`

## Testes

- Sem repositórios in-memory novos — reusar os já existentes de `F1-E01`/`F1-E02`/`F1-E03`/`F1-E04`/`F1-E06` nos unit specs.
- Unit specs: agregação correta por filtro em cada `Generate*ReportUseCase` (dado um conjunto de dados via repositórios in-memory populados, o relatório retorna as linhas esperadas), `ExportCensoEscolarUseCase` sinaliza alunos com campo obrigatório faltante sem travar os demais.
- E2E: um `.e2e-spec.ts` por controller, validando `Content-Type`/`Content-Disposition` do arquivo retornado para cada `format`.

## Definition of Done

- [ ] Critérios de aceite de `F1-E07-U01` e `F1-E07-U02` no BACKLOG satisfeitos.
- [ ] Todos os use cases e controllers implementados com testes unitários e e2e passando.
- [ ] Layout de campos do Censo Escolar/INEP validado contra a especificação oficial vigente (remover o `TODO` do código de `ExportCensoEscolarUseCase`).
- [ ] Hub de relatórios com filtros e exportação PDF/planilha funcionando ponta a ponta.
- [ ] `.claude/docs/features/README.md` atualizado: status de `F1-E07` para 🟢 quando implementado.
