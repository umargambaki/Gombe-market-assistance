# Gombe-market-assistance
 A bilingual Hausa-English business assistant for traders in Gombe, Nigeria

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gombe Market Assistant</title>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #f3f6f4;
      color: #19352b;
    }
    header {
      background: #155b43;
      color: white;
      padding: 22px 18px;
    }
    header h1 { margin: 0 0 8px; font-size: 23px; }
    header p { margin: 0; }
    main { max-width: 900px; margin: auto; padding: 18px; }
    .top {
      display: flex; justify-content: space-between;
      align-items: center; gap: 12px; flex-wrap: wrap;
    }
    button, select, input {
      font: inherit; padding: 11px; border-radius: 8px;
    }
    button {
      cursor: pointer; border: 0;
      background: #155b43; color: white;
    }
    .cards {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(145px, 1fr));
      gap: 12px; margin: 18px 0;
    }
    .card, .panel {
      background: white; border-radius: 12px;
      padding: 16px; box-shadow: 0 2px 8px #18352b10;
    }
    .card p { color: #617269; margin: 0 0 10px; }
    .card strong { font-size: 20px; overflow-wrap: anywhere; }
    .panel { margin-bottom: 18px; }
    h2 { font-size: 19px; margin-top: 0; }
    form { display: grid; gap: 12px; }
    label { display: grid; gap: 6px; }
    input, select {
      width: 100%; border: 1px solid #ccd8d1;
      background: white; color: #19352b;
    }
    .two { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
    .entry {
      border-bottom: 1px solid #e2e9e4;
      padding: 12px 0; display: flex;
      justify-content: space-between; gap: 10px;
    }
    .entry small { display: block; color: #617269; margin-top: 5px; }
    .delete { background: #a52b2b; padding: 7px 10px; }
    .note { font-size: 13px; color: #617269; line-height: 1.5; }
    @media(max-width: 420px) {
      .two { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
  <header>
    <div class="top">
      <div>
        <h1>Gombe Market Assistant</h1>
        <p id="subtitle">Your small business, better managed.</p>
      </div>
      <select id="language" aria-label="Language">
        <option value="en">English</option>
        <option value="ha">Hausa</option>
      </select>
    </div>
  </header>

  <main>
    <h2 id="overview">Business overview</h2>

    <section class="cards">
      <div class="card">
        <p id="salesLabel">Total sales</p>
        <strong id="sales">₦0.00</strong>
      </div>
      <div class="card">
        <p id="expenseLabel">Total expenses</p>
        <strong id="expenses">₦0.00</strong>
      </div>
      <div class="card">
        <p id="balanceLabel">Sales minus expenses</p>
        <strong id="balance">₦0.00</strong>
      </div>
    </section>

    <section class="panel">
      <h2 id="recordTitle">Record a transaction</h2>
      <form id="transactionForm">
        <div class="two">
          <label>
            <span id="typeLabel">Transaction type</span>
            <select id="type">
              <option value="sale">Sale</option>
              <option value="expense">Expense</option>
            </select>
          </label>
          <label>
            <span id="amountLabel">Amount (₦)</span>
            <input id="amount" type="number" min="0.01"
              step="0.01" placeholder="e.g. 5000" required>
          </label>
        </div>
        <label>
          <span id="descriptionLabel">Description</span>
          <input id="description" maxlength="100"
            placeholder="e.g. Phone charger sales" required>
        </label>
        <button id="saveButton" type="submit">Save transaction</button>
      </form>
    </section>

    <section class="panel">
      <h2 id="historyTitle">Transaction history</h2>
      <div id="history"></div>
      <p id="empty" class="note">No transactions yet.</p>
    </section>

    <p id="privacy" class="note">
      Prototype: records are stored only in this browser.
      Clearing browser data may erase them. This is not yet
      a secure accounting system.
    </p>
  </main>

  <script>
    const translations = {
      en: {
        subtitle: "Your small business, better managed.",
        overview: "Business overview",
        salesLabel: "Total sales",
        expenseLabel: "Total expenses",
        balanceLabel: "Sales minus expenses",
        recordTitle: "Record a transaction",
        typeLabel: "Transaction type",
        sale: "Sale",
        expense: "Expense",
        amountLabel: "Amount (₦)",
        descriptionLabel: "Description",
        placeholder: "e.g. Phone charger sales",
        saveButton: "Save transaction",
        historyTitle: "Transaction history",
        empty: "No transactions yet.",
        delete: "Delete",
        privacy: "Prototype: records are stored only in this browser. Clearing browser data may erase them. This is not yet a secure accounting system.",
        confirm: "Delete this transaction?"
      },
      ha: {
        subtitle: "Ka sarrafa kananan kasuwancinka da kyau.",
        overview: "Takaitaccen kasuwanci",
        salesLabel: "Jimillar tallace-tallace",
        expenseLabel: "Jimillar kashe-kashe",
        balanceLabel: "Tallace-tallace bayan kashe-kashe",
        recordTitle: "Rubuta ciniki",
        typeLabel: "Nau'in ciniki",
        sale: "Sayarwa",
        expense: "Kashe kudi",
        amountLabel: "Adadi (₦)",
        descriptionLabel: "Bayani",
        placeholder: "Misali: Sayar da cajar waya",
        saveButton: "Ajiye ciniki",
        historyTitle: "Tarihin cinikayya",
        empty: "Babu ciniki tukuna.",
        delete: "Goge",
        privacy: "Wannan samfurin gwaji yana ajiye bayanai a wannan burauzar kawai. Share bayanan burauza na iya goge su. Ba cikakken tsarin lissafi mai tsaro ba ne tukuna.",
        confirm: "A goge wannan ciniki?"
      }
    };

    const $ = id => document.getElementById(id);
    let records = [];

    try {
      records = JSON.parse(
        localStorage.getItem("gombeTransactions") || "[]"
      );
      if (!Array.isArray(records)) records = [];
    } catch {
      records = [];
    }

    let lang = "en";

    function money(value) {
      return new Intl.NumberFormat("en-NG", {
        style: "currency",
        currency: "NGN"
      }).format(value);
    }

    function save() {
      localStorage.setItem(
        "gombeTransactions", JSON.stringify(records)
      );
    }

    function render() {
      const sales = records
        .filter(r => r.type === "sale")
        .reduce((sum, r) => sum + r.amount, 0);

      const expenses = records
        .filter(r => r.type === "expense")
        .reduce((sum, r) => sum + r.amount, 0);

      $("sales").textContent = money(sales);
      $("expenses").textContent = money(expenses);
      $("balance").textContent = money(sales - expenses);

      $("history").replaceChildren();
      $("empty").hidden = records.length > 0;

      [...records].reverse().forEach(record => {
        const entry = document.createElement("div");
        entry.className = "entry";

        const details = document.createElement("div");
        const name = document.createElement("strong");
        name.textContent = record.description;

        const info = document.createElement("small");
        info.textContent =
          T[record.type] + " · " +
          new Date(record.date).toLocaleDateString();

        details.append(name, info);

        const right = document.createElement("div");
        const amount = document.createElement("strong");
        amount.textContent = money(record.amount);

        const del = document.createElement("button");
        del.className = "delete";
        del.textContent = T.delete;
        del.type = "button";
        del.addEventListener("click", () => {
          if (confirm(T.confirm)) {
            records = records.filter(r => r.id !== record.id);
            save();
            render();
          }
        });

        right.append(amount, document.createElement("br"), del);
        entry.append(details, right);
        $("history").append(entry);
      });
    }

    let T = translations.en;

    function setLanguage(language) {
      lang = language;
      T = translations[lang];
      document.documentElement.lang = lang;

      [
        "subtitle", "overview", "salesLabel", "expenseLabel",
        "balanceLabel", "recordTitle", "typeLabel",
        "amountLabel", "descriptionLabel", "saveButton",
        "historyTitle", "privacy"
      ].forEach(id => {
        $(id).textContent = T[id];
      });

      $("type").options[0].textContent = T.sale;
      $("type").options[1].textContent = T.expense;
      $("description").placeholder = T.placeholder;
      $("empty").textContent = T.empty;
      render();
    }

    $("language").addEventListener("change", event => {
      setLanguage(event.target.value);
    });

    $("transactionForm").addEventListener("submit", event => {
      event.preventDefault();

      const amount = Number($("amount").value);
      const description = $("description").value.trim();

      if (!Number.isFinite(amount) || amount <= 0 || !description) {
        alert("Enter a valid amount and description.");
        return;
      }

      records.push({
        id: Date.now() + Math.random(),
        type: $("type").value,
        amount,
        description,
        date: new Date().toISOString()
      });

      try {
        save();
      } catch {
        records.pop();
        alert("Could not save. Check your browser storage.");
        return;
      }

      $("transactionForm").reset();
      render();
    });

    setLanguage("en");
  </script>
</body>
</html>
