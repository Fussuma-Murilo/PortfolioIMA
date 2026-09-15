# IMA+ — Diagrama de classes

Diagrama único, gerado a partir de `src/main/java`. Usa os **nomes reais** do
código; ver "Convenção de nomes", "Critério de seleção" e "Divergências" ao final.

```mermaid
classDiagram
    direction TB

    %% ───────────────────────── Controllers ─────────────────────────
    class InvoiceController {
        +create(request: InvoiceDTO, user: UserEntity) InvoiceDTO
        +findAll() List~InvoiceDTO~
        +findById(id: Long) InvoiceDTO
        +update(id: Long, request: InvoiceDTO, user: UserEntity) InvoiceDTO
        +delete(id: Long, user: UserEntity) void
        +generateChaveAcesso(request: InvoiceDTO) Map
        +validateChaveAcesso(request: InvoiceDTO) Map
        +exportXml(id: Long, origem: String) byte[]
    }
    class PartnerController {
        +findAll(termo: String) List~PartnerDTO~
        +findById(id: Long) PartnerDTO
        +create(request: PartnerDTO) PartnerDTO
        +update(id: Long, request: PartnerDTO) PartnerDTO
        +delete(id: Long) void
    }
    class XmlImportController {
        +importXml(file: MultipartFile, user: UserEntity) InvoiceDTO
    }
    class InvoiceAiValidationController {
        +getLatest(id: Long) AiValidationDTO
        +request(id: Long, user: UserEntity) AiValidationDTO
    }
    class InvoiceExportController {
        +exportXlsx(dataInicio: LocalDate, dataFim: LocalDate, idParceiro: Long, tipoDocumento: String, documentStatus: String) byte[]
    }
    class NcmImportController {
        +preview(file: MultipartFile) NcmDiffResultDTO
        +importFile(file: MultipartFile) NcmImportResultDTO
    }
    class AuthenticationController {
        +login(credentials: AuthenticationDTO) LoginResponseDTO
        +register(userRecord: UserEntity) void
    }

    %% ───────────────────────── Serviços ─────────────────────────
    class InvoiceService {
        +findAll() List~InvoiceDTO~
        +findById(id: Long) InvoiceDTO
        +save(dto: InvoiceDTO, idUsuario: Long) InvoiceDTO
        +update(id: Long, dto: InvoiceDTO, idUsuario: Long) InvoiceDTO
        +delete(id: Long, userId: Long) void
        +validateChaveAcesso(dto: InvoiceDTO) Map
        +generateChaveAcesso(dto: InvoiceDTO) String
        +toDTO(entity: InvoiceEntity) InvoiceDTO
        -requestAiValidation(idDocumentoFiscal: Long, origin: AiValidationOrigin, idUsuario: Long) void
    }
    class PartnerService {
        +findAll(termo: String) List~PartnerDTO~
        +findById(id: Long) PartnerDTO
        +save(dto: PartnerDTO) PartnerDTO
        +update(id: Long, dto: PartnerDTO) PartnerDTO
        +delete(id: Long) void
    }
    class XmlImportService {
        +importXml(file: MultipartFile, idUsuarioLogado: Long) InvoiceEntity
        -reactivateExistingInvoice(idDocumentoFiscal: Long, idUsuarioLogado: Long) InvoiceEntity
        -checkChaveAgainstXml(doc: Document, chaveAcesso: String) void
        -resolveOrCreatePartner(doc: Document, tagGroup: String) PartnerEntity
    }
    class XmlReaderService {
        +parse(xmlStream: InputStream) InvoiceEntity
    }
    class NfeXmlExportService {
        +export(idDocumentoFiscal: Long, forcarGeracao: boolean) ExportedXml
        +generate(nota: InvoiceEntity) String
    }
    class ChaveAcessoService {
        +validate(chaveInformada: String, nota: InvoiceReference) Map
        +requireValid(chaveInformada: String, nota: InvoiceReference) void
        +generate(nota: InvoiceReference) ChaveAcesso
        +checkAgainstXml(chaveInformada: String, cUF: String, dhEmi: String, documentoEmitente: String, mod: String, serie: String, nNF: String, tpEmis: String, cNF: String, cDV: String) List~String~
    }
    class InvoiceExportService {
        <<interface>>
        +export(filtro: InvoiceExportFilter) ByteArrayOutputStream
    }
    class InvoiceExportServiceImpl {
        +export(filtro: InvoiceExportFilter) ByteArrayOutputStream
    }

    %% ───────────────────── Validação por IA ─────────────────────
    class AiValidationService {
        +request(idDocumentoFiscal: Long, origin: AiValidationOrigin, idUsuario: Long) Optional~AiValidationEntity~
        +enqueue(idDocumentoFiscal: Long, origin: AiValidationOrigin, idUsuario: Long) Optional~AiValidationEntity~
        +markProcessing(idValidacao: Long) boolean
        +complete(idValidacao: Long, resultado: InvoiceAiValidationResultDTO, rawResponse: String, aiModel: String) void
        +fail(idValidacao: Long, errorMessage: String, rawResponse: String) void
        +requeue(idValidacao: Long) void
        +findLatest(idDocumentoFiscal: Long) Optional~AiValidationDTO~
    }
    class AiValidationRequestedEvent {
        <<record>>
        +Long idValidacao
    }
    class AiValidationProcessor {
        +onRequested(evento: AiValidationRequestedEvent) void
        +process(idValidacao: Long) void
        +recoverPendingJobs() void
    }
    class InvoiceAiValidationService {
        +buildPayload(idDocumentoFiscal: Long) AnonymizedInvoiceDTO
        +callAi(payload: AnonymizedInvoiceDTO) String
        +parseResponse(rawResponse: String, idDocumentoFiscal: Long) InvoiceAiValidationResultDTO
    }
    class OpenAiClient {
        +requestValidation(systemPrompt: String, userPrompt: String, jsonSchema: JsonNode) String
    }

    %% ──────────────── NCM: leitura por estratégia ────────────────
    class NcmImportService {
        +importFile(file: MultipartFile) NcmImportResultDTO
    }
    class NcmDiffService {
        +preview(file: MultipartFile) NcmDiffResultDTO
    }
    class NcmLookupService {
        +findExistingByCode(codes: Set~String~) Map~String,NcmEntity~
        +findActiveMissingFrom(codesInFile: Set~String~) List~NcmEntity~
    }
    class NcmValidationService {
        +validate(codigoNcm: String, dataReferencia: LocalDate) NcmStructuralCheckResult
    }
    class NcmSpreadsheetReader {
        -List~NcmFileReader~ readers
        +read(file: MultipartFile) NcmSpreadsheetReadResult
    }
    class NcmFileReader {
        <<interface>>
        +supports(file: MultipartFile) boolean
        +read(file: MultipartFile) NcmSpreadsheetReadResult
    }
    class NcmXlsxReader
    class NcmCsvReader

    %% ───────────────────────── Segurança ─────────────────────────
    class SecurityConfig {
        +securityFilterChain(http: HttpSecurity) SecurityFilterChain
        +authenticationManager(authenticationConfiguration: AuthenticationConfiguration) AuthenticationManager
        +passwordEncoder() PasswordEncoder
    }
    class SecurityFilter {
        #doFilterInternal(request: HttpServletRequest, response: HttpServletResponse, filterChain: FilterChain) void
        -recoverToken(request: HttpServletRequest) String
    }
    class AuditFilter {
        #doFilterInternal(request: HttpServletRequest, response: HttpServletResponse, filterChain: FilterChain) void
    }
    class TokenService {
        +generateToken(user: UserEntity) String
        +validateToken(token: String) String
    }
    class AuthenticationService {
        +loadUserByUsername(username: String) UserDetails
    }

    %% ───────────────────────── Repositórios ─────────────────────────
    class InvoiceRepository {
        <<interface>>
        +existsByChaveAcessoAndActiveTrue(chaveAcesso: String) boolean
        +findInactiveIdByChaveAcesso(chave: String) Optional~Number~
        +reactivate(id: Long) int
        +existsByLinkedPartner(idParceiro: Long) boolean
    }
    class PartnerRepository {
        <<interface>>
        +findByDocument(document: String) Optional~PartnerEntity~
        +existsByDocument(document: String) boolean
        +findByActiveTrueOrderByLegalNameAsc() List~PartnerEntity~
        +searchByTerm(termo: String) List~PartnerEntity~
    }
    class AiValidationRepository {
        <<interface>>
        +findWithFindings(id: Long) Optional~AiValidationEntity~
        +existsByIdDocumentoFiscalAndProcessingStatusIn(idDocumentoFiscal: Long, status: Collection~AiValidationStatus~) boolean
        +findPendingOrStalled(limite: LocalDateTime, limit: Limit) List~AiValidationEntity~
    }
    class XmlFileRepository {
        <<interface>>
        +findByContentHash(contentHash: String) Optional~XmlFileEntity~
    }
    class UserRepository {
        <<interface>>
        +findByLogin(login: String) UserDetails
    }
    class NcmRepository {
        <<interface>>
        +findByCode(code: String) Optional~NcmEntity~
        +findByCodeIn(codes: Collection~String~) List~NcmEntity~
        +findByActiveTrue() List~NcmEntity~
    }

    %% ───────────────────── Tratamento de erro ─────────────────────
    class GlobalExceptionHandler {
        +handleNotFound(e: ResourceNotFoundException, request: HttpServletRequest) ApiErrorDTO
        +handleBusinessRule(e: BusinessException, request: HttpServletRequest) ApiErrorDTO
        +handleInvalidData(e: InvalidDataException, request: HttpServletRequest) ApiErrorDTO
        +handleInvalidFields(e: InvalidFieldsException, request: HttpServletRequest) ApiErrorDTO
        +handleUnexpectedError(e: Exception, request: HttpServletRequest) ApiErrorDTO
    }
    class ResourceNotFoundException
    class BusinessException
    class InvalidDataException
    class InvalidFieldsException {
        -Map~String,String~ campos
    }

    %% ───────────────────────── Entidades ─────────────────────────
    class InvoiceEntity {
        +Long idDocumentoFiscal
        +Boolean active
        +String tipoDocumento
        +String chaveAcesso
        +String numero
        +String serie
        +String modelo
        +LocalDateTime issueDate
        +LocalDateTime inOutDate
        +String naturezaOperacao
        +String tipoOperacao
        +String documentStatus
        +BigDecimal grossAmount
        +BigDecimal discountAmount
        +BigDecimal freightAmount
        +BigDecimal insuranceAmount
        +BigDecimal otherAmount
        +BigDecimal taxAmount
        +BigDecimal totalAmount
        +BigDecimal icmsAmount
        +BigDecimal ipiAmount
        +BigDecimal pisAmount
        +BigDecimal cofinsAmount
        +Long idArquivoXml
        +LocalDateTime createdAt
        +Long createdByUserId
        +LocalDateTime updatedAt
        +Long updatedByUserId
        +addItem(item: InvoiceItemEntity) void
        +removeItem(item: InvoiceItemEntity) void
    }
    class InvoiceItemEntity {
        +Long idDocumentoFiscalItem
        +Integer itemNumber
        +String productCode
        +String productDescription
        +String ncm
        +String cfop
        +String commercialUnit
        +BigDecimal quantity
        +BigDecimal unitAmount
        +BigDecimal grossAmount
        +BigDecimal discountAmount
        +BigDecimal freightAmount
        +BigDecimal insuranceAmount
        +BigDecimal otherAmount
        +BigDecimal baseCalculoIcms
        +BigDecimal aliquotaIcms
        +BigDecimal icmsAmount
        +BigDecimal baseCalculoIpi
        +BigDecimal aliquotaIpi
        +BigDecimal ipiAmount
        +BigDecimal baseCalculoPis
        +BigDecimal aliquotaPis
        +BigDecimal pisAmount
        +BigDecimal baseCalculoCofins
        +BigDecimal aliquotaCofins
        +BigDecimal cofinsAmount
        +String cstIcms
        +String cstPis
        +String cstCofins
    }
    class InvoiceTransportEntity {
        +Long idTransporteDocumento
        +String modalidadeFrete
        +Integer volumes
        +String packageType
        +String brand
        +String numbering
        +BigDecimal grossWeight
        +BigDecimal netWeight
        +String vehiclePlate
        +String ufPlaca
    }
    class InvoicePaymentEntity {
        +Long idPagamentoDocumento
        +String paymentMethod
        +BigDecimal paymentAmount
        +Integer installmentCount
        +BigDecimal installmentAmount
        +BigDecimal totalInstallmentAmount
        +Boolean uniformInstallments
    }
    class PartnerEntity {
        +Long idParceiro
        +String tipoParceiro
        +String legalName
        +String tradeName
        +String document
        +String tipoDocumento
        +String inscrEstadual
        +String inscrMunicipal
        +String country
        +String uf
        +String city
        +String cep
        +String street
        +String numero
        +String complement
        +String district
        +String email
        +String phone
        +Boolean active
    }
    class XmlFileEntity {
        +Long idArquivoXml
        +String fileName
        +String filePath
        +String contentHash
        +String xmlContent
        +String tipoDocumento
        +LocalDateTime importedAt
        +Long importedByUserId
        +String processingStatus
        +String errorMessage
    }
    class AiValidationEntity {
        +Long idValidacao
        +Long idDocumentoFiscal
        +AiValidationStatus processingStatus
        +String overallStatus
        +AiValidationOrigin origin
        +LocalDateTime requestedAt
        +LocalDateTime completedAt
        +String errorMessage
        +Long requestedByUserId
        +Integer attempts
        +String aiModel
        +String rawResponse
        +addFinding(achado: AiValidationFindingEntity) void
    }
    class AiValidationFindingEntity {
        +Long idAchado
        +Integer sortOrder
        +String scope
        +Integer itemNumber
        +String field
        +String severity
        +String message
        +String suggestion
    }
    class UserEntity {
        +Long idUser
        +String name
        +String email
        +String login
        +String password
        +Boolean active
        +LocalDateTime creationDate
        +Roles role
        +getAuthorities() Collection
    }
    class NcmEntity {
        +Long id
        +String code
        +String description
        +LocalDate startDate
        +LocalDate endDate
        +Boolean active
    }

    %% ───────────────────────── Enumerações ─────────────────────────
    class PartnerType {
        <<enumeration>>
        CLIENTE
        FORNECEDOR
        TRANSPORTADORA
        OUTROS
    }
    class AiValidationStatus {
        <<enumeration>>
        PENDENTE
        PROCESSANDO
        PROCESSADO
        ERRO
    }
    class AiValidationOrigin {
        <<enumeration>>
        IMPORTACAO_XML
        CRIACAO
        ATUALIZACAO
        MANUAL
    }
    class Roles {
        <<enumeration>>
        JURIDICO
        OPERACIONAL
    }

    %% ───────────────────── Composição do domínio ─────────────────────
    InvoiceEntity "1" *-- "0..*" InvoiceItemEntity : items
    InvoiceEntity "1" *-- "0..1" InvoiceTransportEntity : transport
    InvoiceEntity "1" *-- "0..1" InvoicePaymentEntity : payment
    InvoiceEntity "0..*" --> "1" PartnerEntity : emitente
    InvoiceEntity "0..*" --> "0..1" PartnerEntity : destinatario
    InvoiceEntity "0..*" --> "0..1" PartnerEntity : transportadora
    InvoiceEntity ..> XmlFileEntity : idArquivoXml
    AiValidationEntity "1" *-- "0..*" AiValidationFindingEntity : findings
    AiValidationEntity ..> InvoiceEntity : idDocumentoFiscal
    AiValidationEntity --> AiValidationStatus
    AiValidationEntity --> AiValidationOrigin
    PartnerEntity ..> PartnerType : validado contra
    UserEntity --> Roles

    %% ───────────────────── Controller → Serviço ─────────────────────
    InvoiceController --> InvoiceService
    InvoiceController --> NfeXmlExportService
    PartnerController --> PartnerService
    XmlImportController --> XmlImportService
    XmlImportController --> InvoiceService : toDTO
    InvoiceAiValidationController --> AiValidationService
    InvoiceExportController --> InvoiceExportService
    NcmImportController --> NcmImportService
    NcmImportController --> NcmDiffService
    AuthenticationController --> TokenService
    AuthenticationController --> UserRepository

    %% ───────────────────── Serviço → Serviço / Repositório ─────────────────────
    InvoiceService --> InvoiceRepository
    InvoiceService --> PartnerRepository
    InvoiceService --> ChaveAcessoService
    InvoiceService --> AiValidationService : requestAiValidation
    PartnerService --> PartnerRepository
    PartnerService --> InvoiceRepository : existsByLinkedPartner
    XmlImportService --> XmlReaderService
    XmlImportService --> ChaveAcessoService : checkAgainstXml
    XmlImportService --> InvoiceRepository
    XmlImportService --> XmlFileRepository
    XmlImportService --> PartnerRepository
    XmlImportService --> AiValidationService
    NfeXmlExportService --> InvoiceRepository
    NfeXmlExportService --> XmlFileRepository
    InvoiceExportService <|.. InvoiceExportServiceImpl
    InvoiceExportServiceImpl --> InvoiceRepository

    %% ───────────────────── Fila de validação por IA ─────────────────────
    AiValidationService --> AiValidationRepository
    AiValidationService ..> AiValidationRequestedEvent : publica
    AiValidationRequestedEvent ..> AiValidationProcessor : AFTER_COMMIT
    AiValidationProcessor --> AiValidationService
    AiValidationProcessor --> InvoiceAiValidationService
    AiValidationProcessor --> AiValidationRepository : jobs travados
    InvoiceAiValidationService --> InvoiceRepository
    InvoiceAiValidationService --> OpenAiClient
    InvoiceAiValidationService ..> NcmValidationService : via InvoiceAnonymizerMapper

    %% ───────────────────── Padrão Strategy (NCM) ─────────────────────
    NcmImportService --> NcmSpreadsheetReader
    NcmImportService --> NcmLookupService
    NcmImportService --> NcmRepository
    NcmDiffService --> NcmSpreadsheetReader
    NcmDiffService --> NcmLookupService
    NcmLookupService --> NcmRepository
    NcmValidationService --> NcmRepository
    NcmFileReader <|.. NcmXlsxReader
    NcmFileReader <|.. NcmCsvReader
    NcmSpreadsheetReader o-- "1..*" NcmFileReader : readers

    %% ───────────────────── Segurança / erro ─────────────────────
    SecurityConfig --> SecurityFilter
    SecurityConfig --> AuditFilter
    SecurityFilter --> TokenService
    SecurityFilter --> UserRepository
    AuthenticationService --> UserRepository
    AuthenticationService ..> UserEntity
    InvalidDataException <|-- InvalidFieldsException
    GlobalExceptionHandler ..> ResourceNotFoundException
    GlobalExceptionHandler ..> BusinessException
    GlobalExceptionHandler ..> InvalidDataException
    GlobalExceptionHandler ..> InvalidFieldsException

    %% ───────────────────── Repositório → Entidade ─────────────────────
    InvoiceRepository ..> InvoiceEntity
    PartnerRepository ..> PartnerEntity
    AiValidationRepository ..> AiValidationEntity
    XmlFileRepository ..> XmlFileEntity
    UserRepository ..> UserEntity
    NcmRepository ..> NcmEntity
```

---

## Convenção de nomes

Identificadores seguem inglês para vocabulário **genérico** — verbos (`save`,
`findById`, `update`, `delete`), metadados de auditoria (`createdAt`,
`createdByUserId`), valores (`totalAmount`, `grossWeight`) e endereço (`city`,
`street`, `district`).

Permanecem em português os termos da **nomenclatura oficial da NF-e**, que não
têm equivalente útil em inglês e precisam casar com a documentação do SEFAZ:

`chaveAcesso`, `naturezaOperacao`, `tipoOperacao`, `tipoDocumento`, `cfop`,
`ncm`, `cstIcms`, `cstPis`, `cstCofins`, `aliquotaIcms`, `baseCalculoIcms`
(e equivalentes de IPI/PIS/COFINS), `inscrEstadual`, `inscrMunicipal`,
`modalidadeFrete`, `emitente`, `destinatario`, `transportadora`, `tipoParceiro`,
`serie`, `modelo`, `numero`, `uf`, `cep`.

A regra vale também para os métodos: o verbo é inglês e o substantivo fiscal
fica em português — `validateChaveAcesso`, `generateChaveAcesso`,
`checkChaveAgainstXml`, `writeEmitente`, `resolveInscrEstadual`. Traduzir
`chaveAcesso` para `accessKey` afastaria o código do vocabulário fiscal e
dificultaria a conferência contra o layout da nota.

**Os nomes de coluna no banco continuam em português** (`chave_acesso`,
`valor_total`, `documento`). O vínculo é feito por `@Column(name = "...")`
explícito em cada campo renomeado, de modo que a padronização não exigiu
migração de schema.

---

## Critério de seleção

O código tem 91 tipos; o diagrama mostra 58. Entram as classes que carregam
**decisão de arquitetura ou regra de domínio**: controllers, serviços,
repositórios, entidades, enums, a cadeia de segurança, o tratamento de erro,
a fila de validação por IA e o Strategy de leitura de NCM.

Ficam de fora, de propósito:

- **DTOs** (17 records/classes) — contratos de API que apenas espelham as
  entidades. Numa visão única afogariam o domínio; se forem necessários, pedem
  um segundo diagrama só de contratos.
- **Colaboradores internos de um único serviço** — `InvoiceAnonymizerMapper`,
  `InvoiceValidationPromptFactory` (IA), `InvoiceSpecifications` (exportação
  XLSX), `NcmRowSupport` e os records `ChaveAcesso`, `InvoiceReference`,
  `ExportedXml`, `NcmStructuralCheckResult`. Onde a colaboração cruza módulos
  (IA → NCM), a aresta aparece com o rótulo "via InvoiceAnonymizerMapper".
- **Utilitários estáticos e enums de tabela fiscal** — `DocumentoFederal`,
  `UfIbge`, `SerieReservada`.
- **Configuração** — `AsyncConfig`, `JacksonConfig`, `OpenAiProperties`,
  `ImaplusApplication`.
- **Exceções de integração** — `AiValidationException` e
  `OpenAiIntegrationException`; ambas são traduzidas para HTTP 502 pelo
  `GlobalExceptionHandler`.

---

## Notas de leitura

**Ligações tracejadas para entidades** (`InvoiceEntity ..> XmlFileEntity`,
`AiValidationEntity ..> InvoiceEntity`): o id é guardado como número solto, sem
`@ManyToOne`. Na validação isso é deliberado — o `@SQLRestriction("ativo = 1")`
de `InvoiceEntity` faria a validação de uma nota excluída desaparecer junto. A
integridade fica por conta da chave estrangeira no banco.

**A validação por IA é assíncrona e persistida.** `InvoiceService`,
`XmlImportService` e `InvoiceAiValidationController` chamam
`AiValidationService.request`, que grava um registro `PENDENTE` e publica
`AiValidationRequestedEvent`. `AiValidationProcessor` não é chamado por nenhum
controller: é acionado pelo Spring via `@TransactionalEventListener(AFTER_COMMIT)`
em thread própria (`validacaoIaExecutor`) e, por `@Scheduled`, varre jobs
pendentes ou travados com `findPendingOrStalled`. Os métodos `markProcessing`,
`complete` e `fail` rodam em `REQUIRES_NEW` para que o estado do job sobreviva
a uma falha na chamada externa.

**`ChaveAcessoService` é compartilhado por dois caminhos de entrada.** No CRUD,
`InvoiceService` chama `requireValid` antes de salvar; na importação,
`XmlImportService` chama `checkAgainstXml` para cruzar a chave com os campos
`ide` do XML. É a única regra de negócio puramente fiscal com serviço próprio.

**`AuthenticationService` não aparece como dependência de ninguém.** Ele
implementa `UserDetailsService` e é descoberto **por tipo** pelo Spring Security
ao montar o `AuthenticationManager`. É essencial, apesar de parecer órfão.

**`InvoiceExportService` é a única interface de serviço.** As demais classes de
serviço são injetadas pelo tipo concreto; a interface existe para isolar a
dependência do Apache POI em `InvoiceExportServiceImpl`.

**`NcmSpreadsheetReader` agrega `NcmFileReader`**: o Spring injeta todas as
implementações numa lista e o leitor certo é escolhido pela extensão do arquivo
(`supports`). `NcmLookupService` existe para quebrar consultas `IN (...)` em
lotes de 900 códigos, por causa do limite de 1000 do Oracle.

---

## Divergências em relação ao diagrama anterior

O diagrama anterior descreve um sistema **planejado** que o código não implementa.
Nenhuma das classes daquele diagrama existe com aquele nome — o código está em
inglês (`InvoiceEntity`, `PartnerEntity`, `UserEntity`).

| Diagrama anterior | Código real |
|---|---|
| `DocumentoFiscalItem` decomposto em `ValoresItemFiscal`, `ImpostosItemFiscal`, `ProdutoItemFiscal` + hierarquia `ImpostoItemFiscal` | `InvoiceItemEntity` **plano**: os 30 campos estão inline, sem composição nem herança |
| `Role` como classe, com `Usuario.role : Role` | `Roles` é **enum**; não existe tabela de papéis |
| `TipoParticipante` = EMITENTE / DESTINATARIO / TRANSPORTADORA, atributo de `Participante` | `tipoParceiro` = CLIENTE / FORNECEDOR / TRANSPORTADORA / OUTROS. Emitente/destinatário são papéis **por nota**, não atributos do parceiro |
| `UsuarioController` com ativar, desativar, criar, atualizar, listar, verificarRole | `AuthenticationController` só tem `login` e `register` |
| `RoleController`, `ParticipanteController`, `DocumentoFiscalController` | Não existem. Os equivalentes são `PartnerController` e `InvoiceController` |
| — | Ausentes no diagrama anterior: `AiValidationEntity`, `AiValidationFindingEntity`, `XmlFileEntity`, `NcmEntity`, `InvoiceTransportEntity`, o módulo de chave de acesso e toda a exportação XLSX/XML |

A decomposição de itens e a classe `Role` **existiam no código** como POJOs sem
anotação JPA e sem nenhuma referência — foram removidas na limpeza. Eram a
implementação abandonada daquele desenho.
