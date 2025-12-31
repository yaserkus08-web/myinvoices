# myinvoices
myinvoices – Fast, professional invoices in PDF. Free trial, one-time payment for unlimited use.
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>myinvoices | Rechnungen erstellen</title>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500&display=swap" rel="stylesheet">
<style>
body { font-family:'Roboto',sans-serif; margin:0; padding:0; background:#f4f6f8; color:#333; }
header { background:#1976d2; color:white; padding:20px; text-align:center; }
header h1 { margin:0; font-weight:500; }
.lang-toggle { margin-top:8px; cursor:pointer; font-size:0.9em; text-decoration:underline; }
main { max-width:700px; margin:30px auto; background:white; padding:25px; border-radius:10px; box-shadow:0 6px 20px rgba(0,0,0,0.1); }
h2 { margin-top:0; }
label { display:block; margin-top:15px; font-weight:500; }
input, textarea, button { width:100%; padding:12px; margin-top:5px; border-radius:6px; border:1px solid #ccc; font-size:1em; }
textarea { resize:vertical; }
button { background:#1976d2; color:white; border:none; font-weight:bold; cursor:pointer; transition:0.2s; }
button:hover { background:#135ea0; }
#paymentSection { display:none; margin-top:20px; padding:15px; border:1px solid #1976d2; border-radius:8px; background:#e3f2fd; }
#feedbackSection { margin-top:30px; }
.feedbackItem { border-bottom:1px solid #ddd; padding:10px 0; }
footer { text-align:center; margin:30px 0 10px 0; font-size:0.85em; color:#777; }
.logo-input { margin-top:10px; }
@media(max-width:600px){ main{margin:15px;padding:20px;} }
</style>
</head>
<body>
<header>
  <h1>InvoiceQuick 2.0</h1>
  <div class="lang-toggle" onclick="toggleLang()">Deutsch / English</div>
</header>
<main>
<h2 id="formTitle">Erstelle deine Rechnung / Create Invoice</h2>
<form id="invoiceForm">
<label id="labelCompany">Firmenname / Company Name</label>
<input type="text" id="company" placeholder="Max Mustermann / John Doe" required>

<label id="labelClient">Kunde / Client</label>
<input type="text" id="client" placeholder="Firma / Company" required>

<label id="labelService">Leistung / Service</label>
<input type="text" id="service" placeholder="Beratung / Consulting" required>

<label id="labelAmount">Betrag / Amount (€)</label>
<input type="number" id="amount" placeholder="100" required>

<label id="labelVat">MwSt / VAT (%)</label>
<input type="number" id="vat" value="19" required>

<label id="labelLogo">Logo (optional / optional)</label>
<input type="file" id="logo" class="logo-input" accept="image/*">

<button type="submit" id="btnGenerate">Rechnung erstellen / Generate Invoice</button>
</form>

<div id="paymentSection">
<p id="paymentText">Bezahlung für unbegrenzte Rechnungen / Pay once for unlimited invoices</p>
<button id="btnPay">Jetzt zahlen / Pay 9€</button>
</div>

<!-- Feedback Bereich -->
<div id="feedbackSection">
<h3 id="feedbackTitle">Rückmeldungen / Feedback</h3>
<form id="feedbackForm">
<label id="labelFeedback">Deine Rückmeldung / Your Feedback</label>
<textarea id="feedbackText" placeholder="Dein Feedback..." required></textarea>
<button type="submit" id="btnFeedback">Absenden / Submit Feedback</button>
</form>
<div id="feedbackList"></div>
</div>

</main>
<footer>
&copy; 2025 myinvoice | <a href="#">Impressum / Legal</a> | <a href="#">Datenschutz / Privacy</a>
</footer>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script>
let lang='de';
function toggleLang(){
  lang=lang==='de'?'en':'de';
  document.getElementById('labelCompany').innerText = lang==='de' ? 'Firmenname / Company Name':'Company Name';
  document.getElementById('labelClient').innerText = lang==='de' ? 'Kunde / Client':'Client';
  document.getElementById('labelService').innerText = lang==='de' ? 'Leistung / Service':'Service';
  document.getElementById('labelAmount').innerText = lang==='de' ? 'Betrag / Amount (€)':'Amount (€)';
  document.getElementById('labelVat').innerText = lang==='de' ? 'MwSt / VAT (%)':'VAT (%)';
  document.getElementById('labelLogo').innerText = lang==='de' ? 'Logo (optional / optional)':'Logo (optional)';
  document.getElementById('btnGenerate').innerText = lang==='de' ? 'Rechnung erstellen / Generate Invoice':'Generate Invoice';
  document.getElementById('paymentText').innerText = lang==='de' ? 'Bezahlung für unbegrenzte Rechnungen / Pay once for unlimited invoices':'Pay once for unlimited invoices';
  document.getElementById('btnPay').innerText = lang==='de' ? 'Jetzt zahlen / Pay 9€':'Pay 9€';
  document.getElementById('formTitle').innerText = lang==='de' ? 'Erstelle deine Rechnung / Create Invoice':'Create your Invoice';
  document.getElementById('feedbackTitle').innerText = lang==='de' ? 'Rückmeldungen / Feedback':'Feedback';
  document.getElementById('labelFeedback').innerText = lang==='de' ? 'Deine Rückmeldung / Your Feedback':'Your Feedback';
  document.getElementById('btnFeedback').innerText = lang==='de' ? 'Absenden / Submit Feedback':'Submit Feedback';
}

let freeUsed=false;

// PDF Rechnung generieren
document.getElementById('invoiceForm').addEventListener('submit', function(e){
  e.preventDefault();
  if(freeUsed){ document.getElementById('paymentSection').style.display='block'; return; }

  const company=document.getElementById('company').value;
  const client=document.getElementById('client').value;
  const service=document.getElementById('service').value;
  const amount=parseFloat(document.getElementById('amount').value);
  const vat=parseFloat(document.getElementById('vat').value);
  const total=amount+(amount*vat/100);

  const { jsPDF }=window.jspdf;
  const doc=new jsPDF();

  // Logo einfügen
  const logoInput=document.getElementById('logo');
  if(logoInput.files[0]){
    const reader=new FileReader();
    reader.onload=function(e){
      doc.addImage(e.target.result,'PNG',20,10,50,20);
      generatePDF(doc,company,client,service,amount,vat,total);
    }
    reader.readAsDataURL(logoInput.files[0]);
  }else{
    generatePDF(doc,company,client,service,amount,vat,total);
  }

  freeUsed=true;
  document.getElementById('paymentSection').style.display='block';
  alert(lang==='de' ? 'Kostenlose Rechnung erstellt. Für unbegrenzte Rechnungen bitte zahlen.' : 'Free invoice created. Please pay for unlimited invoices.');
});

function generatePDF(doc,company,client,service,amount,vat,total){
  const date=new Date();
  const invoiceNo='INV'+date.getTime();
  doc.setFontSize(16);
  doc.text('Invoice / Rechnung',20,50);
  doc.setFontSize(12);
  doc.text(`Rechnungsnummer / Invoice No: ${invoiceNo}`,20,60);
  doc.text(`Datum / Date: ${date.toLocaleDateString()}`,20,70);
  doc.text(`Company / Firma: ${company}`,20,80);
  doc.text(`Client / Kunde: ${client}`,20,90);
  doc.text(`Service / Leistung: ${service}`,20,100);
  doc.text(`Amount / Betrag: €${amount.toFixed(2)}`,20,110);
  doc.text(`VAT / MwSt: ${vat}%`,20,120);
  doc.setFontSize(14);
  doc.text(`Total / Gesamt: €${total.toFixed(2)}`,20,130);
  doc.save('Invoice.pdf');
}

// Stripe Testmodus
document.getElementById('btnPay').addEventListener('click', function(){
  alert('Stripe Checkout würde hier gestartet werden (Testmodus). Später echtes Geld empfangen.');
});

// Feedback-Funktion
const feedbackList=document.getElementById('feedbackList');
document.getElementById('feedbackForm').addEventListener('submit', function(e){
  e.preventDefault();
  const text=document.getElementById('feedbackText').value.trim();
  if(text==='') return;
  const feedbackItem=document.createElement('div');
  feedbackItem.className='feedbackItem';
  feedbackItem.innerText=text;
  feedbackList.prepend(feedbackItem); // Neueste oben
  document.getElementById('feedbackText').value='';
  alert(lang==='de' ? 'Danke für dein Feedback!' : 'Thank you for your feedback!');
});
</script>
