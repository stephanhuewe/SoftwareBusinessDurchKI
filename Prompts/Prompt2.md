Implementierung: Rechnungsverwaltungssystem
Projektstruktur
Das Projekt besteht aus mehreren Komponenten für eine saubere Architektur:
// Projektstruktur
RechnungsSystem/
  ├── Forms/
  │   ├── MainForm.cs
  │   ├── CustomerForm.cs
  │   └── InvoiceForm.cs
  ├── Models/
  │   ├── Customer.cs
  │   └── Invoice.cs
  ├── Data/
  │   └── Database.cs
  └── Services/
      ├── PdfService.cs
      └── EmailService.cs
​
Datenmodelle
// Models/Customer.cs
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string Address { get; set; }
    public string TaxId { get; set; }
}

// Models/Invoice.cs
public class Invoice
{
    public int Id { get; set; }
    public string Number { get; set; }
    public int CustomerId { get; set; }
    public decimal Amount { get; set; }
    public DateTime IssueDate { get; set; }
    public DateTime DueDate { get; set; }
    public bool IsPaid { get; set; }
}
​
Hauptformular
// Forms/MainForm.cs
public partial class MainForm : Form
{
    private Database _db;
    private PdfService _pdfService;
    private EmailService _emailService;

    public MainForm()
    {
        InitializeComponent();
        _db = new Database();
        _pdfService = new PdfService();
        _emailService = new EmailService();
        
        LoadData();
    }

    private void btnNewInvoice_Click(object sender, EventArgs e)
    {
        var invoiceForm = new InvoiceForm();
        if (invoiceForm.ShowDialog() == DialogResult.OK)
        {
            _db.SaveInvoice(invoiceForm.Invoice);
            _pdfService.GenerateInvoicePdf(invoiceForm.Invoice);
            RefreshInvoiceList();
        }
    }

    private void btnTrackPayments_Click(object sender, EventArgs e)
    {
        var unpaidInvoices = _db.GetUnpaidInvoices();
        foreach (var invoice in unpaidInvoices)
        {
            if (invoice.DueDate < DateTime.Now)
            {
                _emailService.SendPaymentReminder(invoice);
            }
        }
    }
}
​
Datenbankanbindung
// Data/Database.cs
public class Database
{
    private string _connectionString = "Server=localhost;Database=Invoicing;Trusted_Connection=True;";
    
    public void SaveInvoice(Invoice invoice)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            var cmd = new SqlCommand(
                "INSERT INTO Invoices (Number, CustomerId, Amount, IssueDate, DueDate) " +
                "VALUES (@Number, @CustomerId, @Amount, @IssueDate, @DueDate)", connection);
            
            cmd.Parameters.AddWithValue("@Number", invoice.Number);
            cmd.Parameters.AddWithValue("@CustomerId", invoice.CustomerId);
            cmd.Parameters.AddWithValue("@Amount", invoice.Amount);
            cmd.Parameters.AddWithValue("@IssueDate", invoice.IssueDate);
            cmd.Parameters.AddWithValue("@DueDate", invoice.DueDate);
            
            cmd.ExecuteNonQuery();
        }
    }
}
​
PDF-Generierung
// Services/PdfService.cs
public class PdfService
{
    public void GenerateInvoicePdf(Invoice invoice)
    {
        using (var document = new Document())
        {
            PdfWriter.GetInstance(document, 
                new FileStream($"invoice_{invoice.Number}.pdf", FileMode.Create));
            
            document.Open();
            // PDF-Inhalt generieren
            document.Add(new Paragraph($"Rechnung Nr: {invoice.Number}"));
            document.Add(new Paragraph($"Betrag: {invoice.Amount:C2}"));
            document.Add(new Paragraph($"Fällig am: {invoice.DueDate:d}"));
            document.Close();
        }
    }
}
​
Installation und Ausführung
Erstellen Sie eine neue SQL-Datenbank mit dem Namen "Invoicing"
Führen Sie das folgende SQL-Script aus:
CREATE TABLE Customers (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(100),
    Email NVARCHAR(100),
    Address NVARCHAR(200),
    TaxId NVARCHAR(50)
);

CREATE TABLE Invoices (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Number NVARCHAR(20),
    CustomerId INT,
    Amount DECIMAL(18,2),
    IssueDate DATETIME,
    DueDate DATETIME,
    IsPaid BIT DEFAULT 0,
    FOREIGN KEY (CustomerId) REFERENCES Customers(Id)
);
​
NuGet-Pakete
System.Data.SqlClient
iTextSharp
Dapper
Sicherheitshinweise
Verwenden Sie immer parametrisierte Abfragen zur Vermeidung von SQL-Injection
Implementieren Sie eine sichere Benutzerauthentifizierung
Verschlüsseln Sie sensible Daten in der Datenbank
Erstellen Sie regelmäßige Backups der Datenbank
Lizenzierung
Das Programm kann als kommerzielle Software vertrieben werden. Empfohlene Preismodelle:
Basic: 29,99€/Monat (bis zu 100 Rechnungen/Monat)
Professional: 49,99€/Monat (bis zu 500 Rechnungen/Monat)
Enterprise: 99,99€/Monat (unbegrenzte Rechnungen)
