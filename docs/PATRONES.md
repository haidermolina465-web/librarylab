# Patrones de diseño implementados

## Tabla resumen

| Patrón | Tipo | Clase(s) | Capa | Pruebas que lo validan |
|---|---|---|---|---|
| Factory Method | Creacional | LoanFactoryExercise (implementa ILoanFactoryExercise) | Application | FactoryMethodExerciseTests.Creates_a_loan_with_the_requested_resource_type |
| Builder | Creacional | LoanReceiptBuilderExercise (implementa ILoanReceiptBuilderExercise) | Application | BuilderExerciseTests.Builds_a_receipt_that_preserves_loan_details |
| Adapter | Estructural | LegacyCatalogAdapterExercise (implementa ILegacyCatalogAdapterExercise) | Application | AdapterExerciseTests.Converts_a_legacy_catalog_record_to_the_library_import_contract |
| Decorator | Estructural | LoanNoticeDecoratorExercise (implementa ILoanNoticeSender) | Application | DecoratorExerciseTests.Sends_the_loan_notice_to_the_configured_sender |
| Facade | Estructural | MemberOverviewFacadeExercise (implementa IMemberOverviewFacadeExercise) | Application | FacadeExerciseTests.Produces_a_member_overview_with_active_loan_information |

## Factory Method (Creacional)

**Archivo:** src/LibraryLab.Application/Exercises/PatternExerciseContracts.cs

Crea un Loan a partir de CreateLoanCommand y ResourceKind. Genera id con Guid.NewGuid() y borrowedAt con DateTimeOffset.UtcNow.

`csharp
public Loan Create(CreateLoanCommand command, ResourceKind resourceKind) =>
    new Loan(Guid.NewGuid(), command.UserId, command.ResourceId, resourceKind, DateTimeOffset.UtcNow, command.DueAt);
`

## Builder (Creacional)

**Archivo:** src/LibraryLab.Application/Exercises/PatternExerciseContracts.cs

Construye un LoanReceipt a partir de loanId, resourceId y dueAt.

`csharp
public LoanReceipt Build(Guid loanId, Guid resourceId, DateTimeOffset dueAt) =>
    new LoanReceipt(loanId, resourceId, dueAt);
`

## Adapter (Estructural)

**Archivo:** src/LibraryLab.Application/Exercises/PatternExerciseContracts.cs

Traduce LegacyCatalogRecord a CatalogImport, generando Id.

`csharp
public CatalogImport Adapt(LegacyCatalogRecord record) =>
    new CatalogImport(Guid.NewGuid(), record.InventoryCode, record.DisplayName, record.Category);
`

## Decorator (Estructural)

**Archivo:** src/LibraryLab.Application/Exercises/PatternExerciseContracts.cs

Envía el aviso al emisor y lo registra en auditoría.

`csharp
public async Task SendAsync(LoanNotice notice, CancellationToken cancellationToken)
{
    await sender.SendAsync(notice, cancellationToken);
    await audit.RecordAsync(notice, cancellationToken);
}
`

## Facade (Estructural)

**Archivo:** src/LibraryLab.Application/Exercises/PatternExerciseContracts.cs

Obtiene el resumen del miembro consultando usuarios y préstamos, filtrando préstamos activos.

`csharp
public async Task<MemberOverview> GetAsync(Guid userId, CancellationToken cancellationToken)
{
    var usuario = await users.FindAsync(userId, cancellationToken);
    var prestamos = await loans.ListAsync(cancellationToken);
    var activos = prestamos.Where(p => p.Status == LoanStatus.Active);
    var recibos = activos.Select(p => new LoanReceipt(p.Id, p.ResourceId, p.DueAt)).ToList();
    return new MemberOverview(usuario.Id, usuario.Name, recibos);
}
`

## Resultado de pruebas

- Tests de patrones: 5/5 correctas (LibraryLab.Patterns.Tests.dll).
- Suite completa: 39/39 correctas.
- No se modificaron las pruebas existentes.
