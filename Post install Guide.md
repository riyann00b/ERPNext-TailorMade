# ⚙️ Post-Installation Steps

This section outlines the essential configurations and setups required after your initial ERPNext installation is complete.

## 1. Custom Doctype Fields Configuration

To enhance data management and product categorization, the following custom fields have been implemented through ERPNext's `Customize Form` feature:

*   **Product Category**: Categorizes products (e.g., "Dress Material", "Readymade Suit").
*   **Color**: Specifies the color variant of the product.
*   **Size**: Defines the size variant of the product.
*   **Encrypted Buying Price**: Stores the buying price securely.
*   **Stitching Available**: Indicates whether stitching services are available for the item.
*   **50% of SSR**: A custom field likely related to pricing calculations (e.g., "50% of Standard Selling Rate").

## 2. Bulk Upload Preparation: Custom Google Sheet

A custom Google Sheet has been designed to facilitate the bulk upload of item data. It includes comprehensive columns and intelligent formulas to ensure data integrity and automate certain values.

### 2.1. Google Sheet Columns

The sheet contains the following columns:

| Column Header                 | Description                                       |
| :---------------------------- | :------------------------------------------------ |
| Brand                         | Brand name of the product.                        |
| Item Name                     | Name of the product.                              |
| Product Category              | Category of the product (e.g., "Dress Material"). |
| Color                         | Color variant of the product.                     |
| Size                          | Size variant of the product.                      |
| Opening Stock                 | Initial stock quantity.                           |
| Valuation Rate                | Cost price or valuation rate.                     |
| Standard Selling Rate         | Standard selling price.                           |
| 50% of SSR                    | Calculated field (see formulas below).            |
| Suit Code                     | Custom code for suits (see formulas below).       |
| HSN/SAC                       | Harmonized System of Nomenclature / Service Accounting Code. |
| Description                   | Detailed description of the product.              |
| Image                         | URL or identifier for the product image.          |
| Encrypted Buying Price        | Securely stored buying price.                     |
| Item Code                     | ERPNext Item Code (auto-generated).               |
| Barcode (Barcodes)            | Barcode value.                                    |
| Barcode Type (Barcodes)       | Type of barcode (e.g., CODE128).                  |
| UOM (Barcodes)                | Unit of Measure for barcode scanning.             |
| Default Unit of Measure       | Standard Unit of Measure.                         |
| Item Group                    | ERPNext Item Group.                               |
| Stitching Available           | Availability of stitching service.                |
| Supplier (Supplier Items)     | Primary supplier details.                         |
| Item Tax Template (Taxes)     | Associated tax template.                          |
| Maximum Net Rate (Taxes)      | Maximum net rate for taxes.                       |
| Minimum Net Rate (Taxes)      | Minimum net rate for taxes.                       |
| Tax Category (Taxes)          | Tax category classification.                      |

### 2.2. Key Formulas

*   **`50% of SSR` Calculation:**
    ```excel
    =ARRAYFORMULA(H2*2)
    ```
    *Note: This formula appears to calculate double the "Standard Selling Rate" (column H), which might represent a Maximum Retail Price or a similar value rather than strictly 50% of SSR. Verify this logic.*

*   **`HSN/SAC` Auto-Assignment:**
    ```excel
    =ARRAYFORMULA(IF(
      C2="Dress Material",
        IF(REGEXMATCH(LOWER(B2&" "&L2),"cotton|lawn|mulmul|muslin|khadi|cambric|poplin"),"63079011",
        IF(REGEXMATCH(LOWER(B2&" "&L2),"silk|banarasi|tussar|chanderi|kanchipuram|muga|dupioni|matka"),"63079012",
        IF(REGEXMATCH(LOWER(B2&" "&L2),"rayon|viscose|jacquard|brocade|dola|chinon|georgette|chiffon|crepe|velvet|organza|satin|net"),"63079013",
        IF(REGEXMATCH(LOWER(B2&" "&L2),"wool|pashmina|linen|jute"),"63079019",
        "630790"  )))),

      IF(OR(C2="Readymade Suit",C2="Kurti",C2="Top"),
        IF(REGEXMATCH(LOWER(B2&" "&L2),"cotton|lawn|mulmul|muslin|khadi|cambric|poplin"),
          IF(C2="Top",
            IF(REGEXMATCH(LOWER(B2&" "&L2),"knit|jersey|stretch"),"610610","620630"),
            IF(REGEXMATCH(LOWER(B2&" "&L2),"knit|jersey|stretch"),"610412","620412")),
        IF(REGEXMATCH(LOWER(B2&" "&L2),"silk|banarasi|tussar|chanderi|kanchipuram|muga|dupioni|matka"),
          IF(C2="Top",
            IF(REGEXMATCH(LOWER(B2&" "&L2),"knit|jersey|stretch"),"610690","620610"),
            IF(REGEXMATCH(LOWER(B2&" "&L2),"knit|jersey|stretch"),"611490","620419")),
        IF(REGEXMATCH(LOWER(B2&" "&L2),"rayon|viscose|jacquard|brocade|dola|chinon|georgette|chiffon|crepe|velvet|organza|satin|net"),
          IF(C2="Top",
            IF(REGEXMATCH(LOWER(B2&" "&L2),"knit|jersey|stretch"),"610620","620640"),
            IF(REGEXMATCH(LOWER(B2&" "&L2),"knit|jersey|stretch"),"611430","620413")),
        IF(REGEXMATCH(LOWER(B2&" "&L2),"wool|pashmina|linen|jute"),
          IF(REGEXMATCH(LOWER(B2&" "&L2),"knit|jersey|stretch"),"611490","621149"),
        IF(REGEXMATCH(LOWER(B2&" "&L2),"knit|jersey|stretch"),"611490","621149")
        )))),

      "" )))
    ```
    *This formula dynamically assigns HSN/SAC codes based on product category, item name, and fabric type.*

*   **`Item Code` Generation:**
    ```excel
    =ARRAYFORMULA(CONCATENATE(
      CHAR(RANDBETWEEN(65,90)),
      SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(TEXT(G2,"0"),"0","Z"),"1","A"),"2","B"),"3","C"),"4","D"),"5","E"),"6","F"),"7","G"),"8","H"),"9","I"),
      CHAR(RANDBETWEEN(65,90))
    ))
    ```
    *Generates a unique Item Code using random characters and a transformed "Standard Selling Rate". Please review this for uniqueness and adherence to your preferred item code structure.*

*   **`Suit Code` Generation:**
    ```excel
    =ARRAYFORMULA(CONCATENATE(
      LEFT(
        REGEXREPLACE(UPPER(REGEXREPLACE(B2,"[^A-Za-z ]","")),"(\b\w)\w*\s*","$1"),
        4
      ),
      "-",
      IF(C2="Readymade Suit","RMS",
        IF(C2="Unstitched Fabric","UF",
          IF(OR(C2="Kurti",C2="Kurtis"),"KT",
            IF(C2="Dress Material","DM",
              IF(C2="Top","TP",
                IF(C2="Accessories","AC",LEFT(UPPER(C2),2))
              )
            )
          )
        )
      ),
      IF(D2<>"",CONCATENATE("-",LEFT(UPPER(D2),1)),""),
      IF(E2<>"",CONCATENATE("-",UPPER(E2)),""),
      "-",
      TEXT(ROW()-1,"00")
    )
    )
    ```
    *Creates a structured "Suit Code" using Item Name, Category, Color, and Size.*

*   **`Product Category` Duplication:**
    ```excel
    =ARRAYFORMULA(C2)
    ```
    *This formula simply duplicates the value from the `Product Category` column (C). Confirm this is the desired functionality.*

## 3. Custom Print Formats

Print formats dictate the layout for printed documents and PDFs. Two custom formats have been developed:

### 3.1. Thermal Receipt (72mm Width)

Designed for standard thermal receipt printers, this format includes advanced features like custom fields and dynamic information.

**HTML Content:**
```html
<!-- ### FINAL, ROBUST 72mm THERMAL RECEIPT (with Custom Field) ### -->
{%- set company = frappe.get_doc("Company", doc.company) if doc.company else None -%}

<div class="receipt">
    <!-- === HEADER === -->
    <header class="receipt-header">
        {% if company and company.company_logo %}
            <img src="{{ company.company_logo }}" class="logo">
        {% endif %}
        <h2 class="company-name">{{ doc.company }}</h2>
        <p class="company-details">
            {% if company %}{{ company.get_formatted("address") }}{% endif %}<br>
            ☎ {% if company and company.phone_no %}{{ company.phone_no }}{% else %}N/A{% endif %}
            {% if company and company.gstin %}<br>GSTIN: {{ company.gstin }}{% endif %}
        </p>
    </header>

    <hr class="separator">

    <!-- === TRANSACTION INFO === -->
    <section class="info-section">
        <div class="info-item"><span>{{ _("Bill No") }}:</span> <span>{{ doc.name }}</span></div>
        <div class="info-item">
            <span>{{ _("Date") }}:</span>
            <span>{% if doc.posting_date %}{{ frappe.format_date(doc.posting_date, "dd/MM/yy") }}{% endif %}</span>
        </div>
        {% if doc.customer %}
        <div class="info-item"><span>{{ _("Customer") }}:</span> <span>{{ doc.customer_name }}</span></div>
        {% endif %}
    </section>

    <hr class="separator-heavy">

    <!-- === ITEMS BODY === -->
    <section class="items-body">
        <div class="items-header-row">
            <span class="item-name-header">{{ _("ITEM") }}</span>
            <span class="item-total-header">{{ _("TOTAL") }}</span>
        </div>

        {% for item in doc.items %}
        <div class="item-entry">
            <div class="item-name">
                {{ item.item_name }}
                {% set item_doc = frappe.get_doc("Item", item.item_code) if item.item_code else None %}
                {% if item_doc and item_doc.has_variant and item_doc.attributes %}
                    <div class="item-variants">
                    {% for attr in item_doc.attributes %}
                        <span>{{ attr.attribute }}: {{ item[frappe.scrub(attr.attribute)] or attr.attribute_value }}</span>
                    {% endfor %}
                    </div>
                {% endif %}
            </div>
            
            <!-- ### NEW FEATURE: "Before Discount" price from custom_50_of_ssr field ### -->
            {% if item.custom_50_of_ssr and item.custom_50_of_ssr > 0 %}
            <div class="before-discount-price">
                <span>{{ _("Before Discount") }}:</span>
                <span>{{ frappe.format_value(item.custom_50_of_ssr, currency=doc.currency) }}</span>
            </div>
            {% endif %}

            <div class="item-calculation">
                <span>{{ frappe.format_value(item.qty) }} x {{ frappe.format_value(item.rate, currency=doc.currency) }}</span>
                <span class="line-total">{{ frappe.format_value(item.amount, currency=doc.currency) }}</span>
            </div>
            
            {% if item.discount_percentage %}
            <div class="discount-calculation">
                <span>{{ _("Discount") }} @ {{ item.discount_percentage }}%</span>
                <span class="line-total">- {{ frappe.format_value(item.discount_amount, currency=doc.currency) }}</span>
            </div>
            <div class="net-rate-calculation">
                <span>{{ _("Net Rate") }}: {{ frappe.format_value(item.net_rate, currency=doc.currency) }}</span>
            </div>
            {% endif %}
        </div>
        {% endfor %}
    </section>

    <hr class="separator-heavy">

    <!-- === TOTALS SECTION === -->
    <section class="totals-section">
        <div class="total-row">
            <span>{{ _("Subtotal") }}</span>
            <span>{{ frappe.format_value(doc.total, currency=doc.currency) }}</span>
        </div>
        {% if doc.discount_amount %}
        <div class="total-row">
            <span>{{ _("Total Discount") }}</span>
            <span>- {{ frappe.format_value(doc.discount_amount, currency=doc.currency) }}</span>
        </div>
        {% endif %}
        {% if doc.taxes %}
        {% for tax in doc.taxes %}
        <div class="total-row tax-row">
            <span>{{ tax.description }}</span>
            <span>{{ frappe.format_value(tax.tax_amount, currency=doc.currency) }}</span>
        </div>
        {% endfor %}
        {% endif %}
        {% if doc.rounding_adjustment %}
        <div class="total-row">
            <span>{{ _("Rounding") }}</span>
            <span>{{ frappe.format_value(doc.rounding_adjustment, currency=doc.currency) }}</span>
        </div>
        {% endif %}
        <div class="grand-total">
            <span>{{ _("GRAND TOTAL") }}</span>
            <span>{{ frappe.format_value(doc.grand_total, currency=doc.currency) }}</span>
        </div>
    </section>

    <hr class="separator">

    <!-- === PAYMENT SECTION === -->
    <section class="payment-section">
        <div class="section-header">{{ _("PAYMENT DETAILS") }}</div>
        {% if doc.payments %}
        {% for payment in doc.payments %}
            <div class="payment-row">
                <span>{{ payment.mode_of_payment }}:</span>
                <span>{{ frappe.format_value(payment.amount, currency=doc.currency) }}</span>
            </div>
        {% endfor %}
        {% set total_paid = doc.payments|sum(attribute='amount') %}
        {% if total_paid > doc.grand_total %}
        <div class="change-due">
            <span>{{ _("CHANGE DUE") }}:</span>
            <span>{{ frappe.format_value(total_paid - doc.grand_total, currency=doc.currency) }}</span>
        </div>
        {% endif %}
        {% endif %}
    </section>

    <!-- === FOOTER === -->
    <footer class="receipt-footer">
        <p class="thank-you">{{ _("Thank you for your business!") }}</p>
        <p class="terms">
            {% if company and company.terms %}{{ company.terms }}{% else %}{{ _("Return/Exchange within 3 days with receipt.") }}{% endif %}
        </p>
        <div class="barcode">
            <svg id="barcode"></svg>
        </div>
        <div class="qrcode">
            <img src="/files/motitaka.svg" alt="Scan QR Code">
        </div>
        <div class="print-timestamp">
            {{ _("Printed") }}: {{ frappe.utils.now_datetime().strftime("%d/%m/%Y %I:%M %p") }}
        </div>
    </footer>
</div>

<!-- Barcode Generation Script -->
<script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.0/dist/JsBarcode.all.min.js"></script>
<script>
    try {
        JsBarcode("#barcode", "{{ doc.name }}", {
            format: "CODE128", displayValue: false, width: 1.5, height: 40, margin: 0
        });
    } catch (e) { /* Barcode generation failed */ }
</script>
```

**CSS for Receipt:**
```css
/* --- Professional 72mm Thermal Receipt CSS --- */
.receipt {
    width: 72mm;
    font-family: 'SF Mono', 'Courier New', monospace;
    font-size: 13px;
    color: #000;
    font-weight: bold;
}
/* --- HEADER & FOOTER --- */
.receipt-header, .receipt-footer { text-align: center; }
.logo { max-width: 100%; max-height: 60px; margin-bottom: 5px; }
.company-name { font-size: 20px; margin: 0; }
.company-details { font-size: 12px; line-height: 1.4; margin: 3px 0; }
.thank-you { font-size: 15px; margin-top: 10px; }
.terms { font-size: 11px; }

/* --- SEPARATORS --- */
hr.separator { border: none; border-top: 1px dashed #000; margin: 8px 0; }
hr.separator-heavy { border: none; border-top: 2px solid #000; margin: 8px 0; }

/* --- TRANSACTION & PAYMENT INFO --- */
.info-section, .payment-section, .totals-section { margin: 10px 0; }
.section-header { text-align: center; font-size: 15px; margin-bottom: 8px; }
.info-item, .total-row, .payment-row { display: flex; justify-content: space-between; margin-bottom: 3px; }
.change-due { display: flex; justify-content: space-between; font-size: 15px; margin-top: 8px; padding-top: 5px; border-top: 1px dashed #000; }

/* --- ITEMS SECTION --- */
.items-header-row { display: flex; justify-content: space-between; font-size: 12px; border-bottom: 1px solid #000; padding-bottom: 4px; margin-bottom: 4px; }
.item-name-header { text-align: left; }
.item-total-header { text-align: right; }
.item-entry { margin-bottom: 10px; }
.item-name { font-size: 14px; }
.item-variants { font-size: 11px; font-weight: normal; margin-top: 2px; }
.item-calculation, .discount-calculation, .net-rate-calculation, .before-discount-price { display: flex; justify-content: space-between; font-size: 13px; padding-left: 10px; }
.net-rate-calculation { justify-content: flex-end; }
.line-total { text-align: right; }
.before-discount-price { color: #333; font-weight: normal; font-style: italic; }

/* --- TOTALS --- */
.grand-total { display: flex; justify-content: space-between; font-size: 18px; border-top: 2px solid #000; margin-top: 8px; padding-top: 5px; }

/* --- BARCODE, QR & TIMESTAMP --- */
.barcode { text-align: center; padding-top: 10px; }
.qrcode { text-align: center; margin-top: 10px; }
.qrcode img { width: 160px; height: 160px; }
.print-timestamp { font-size: 11px; text-align: center; margin-top: 10px; font-weight: normal; }
```

### 3.2. Thermal Label (50mm x 100mm) - "Motitaka Product Label"

This format is optimized for 50mm x 100mm thermal label printers, offering a modern design with clear visual hierarchy.

**HTML Content:**
```html
{% set company_name = doc.company or frappe.db.get_value("Global Defaults", None, "default_company") or "COMPANY" %}
{% set logo = frappe.get_all("File", filters={"attached_to_doctype": "Company", "attached_to_name": company_name}, fields=["file_url"], limit=1) %}
{% set barcode_value = (doc.barcodes and doc.barcodes[0].barcode) or "" %}
{% set show_size = doc.custom_size and (doc.item_group or "")|lower == "readymade suit" %}
{% set currency = doc.currency or frappe.db.get_value("Company", company_name, "default_currency") or "INR" %}
{% set mrp_price = doc.custom_50_of_ssr or 0 %}
{% set sale_price = doc.standard_rate or 0 %}

<div class="tag">

  <div class="tag-header{% if not (logo and logo|length > 0) %} no-logo{% endif %}">
    {% if logo and logo|length > 0 %}
      <img src="{{ logo[0].file_url }}" class="tag-logo" alt="{{ company_name }} Logo">
    {% endif %}
    <div class="tag-company">{{ company_name|upper }}</div>
  </div>

  {% if doc.brand %}
  <div class="tag-box tag-brand">{{ doc.brand|upper }}</div>
  {% endif %}

  <div class="tag-box tag-item">{{ (doc.item_name or "ITEM NAME")|upper }}</div>

  <div class="tag-row">
    {% if doc.custom_color %}
    <div class="tag-cell">
      <div class="cell-label">COLOR</div>
      <div class="cell-value">{{ doc.custom_color|upper }}</div>
    </div>
    {% endif %}

    {% if show_size %}
    <div class="tag-cell">
      <div class="cell-label">SIZE</div>
      <div class="cell-value">{{ doc.custom_size|upper }}</div>
    </div>
    {% endif %}
  </div>

  <div class="tag-row prices">
    <div class="tag-cell">
      <div class="cell-label">MRP</div>
      <div class="cell-value price-strike">{{ frappe.format_value(mrp_price, {"fieldtype":"Currency","options":currency}) }}</div>
    </div>
    <div class="tag-cell dark">
      <div class="cell-label">SALE 50%</div>
      <div class="cell-value">{{ frappe.format_value(sale_price, {"fieldtype":"Currency","options":currency}) }}</div>
    </div>
  </div>

  <div class="barcode">
    {% if barcode_value %}
      <svg id="tag-barcode" class="barcode-svg"></svg>
      <div class="barcode-text">{{ barcode_value }}</div>
    {% else %}
      <div class="no-barcode">NO BARCODE</div>
    {% endif %}
  </div>

  {% if doc.custom_encrypted_buying_price %}
  <div class="tag-footer">{{ doc.custom_encrypted_buying_price|upper }}</div>
  {% endif %}

</div>

<script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.12.1/dist/JsBarcode.all.min.js"></script>
<script>
(function(){
  if (typeof JsBarcode === "undefined") return;
  const value = "{{ barcode_value|e }}".trim();
  if (!value) return;
  const svg = document.getElementById("tag-barcode");
  if (!svg) return;

  JsBarcode(svg, value, {
    format: "CODE128",
    width: 1.4,
    height: 45,
    margin: 0,
    displayValue: false,
    background: "#FFFFFF",
    lineColor: "#000000"
  });

  svg.setAttribute("shape-rendering", "crispEdges");
  svg.style.width = "42mm";
  svg.style.height = "15mm";
})();
</script>
```

**CSS for Label:**
```css
@page {
  size: 50mm 100mm;
  margin: 0;
}

body {
  margin: 0;
  background: #ffffff;
  color: #000000;
  font-family: Arial, Helvetica, sans-serif;
  -webkit-print-color-adjust: exact;
  print-color-adjust: exact;
}

.tag {
  width: 50mm;
  height: 100mm;
  padding: 3mm 2mm;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  gap: 1.6mm;
}

/* Header */
.tag-header {
  border-bottom: 1px solid #000;
  padding: 0 0.6mm 1.1mm;
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 1.2mm;
  text-align: center;
}
.tag-logo {
  max-height: 9mm;
  max-width: 16mm;
  object-fit: contain;
  flex-shrink: 0;
}
.tag-company {
  font-weight: 900;
  font-size: 9.4pt;
  letter-spacing: 0.06em;
  flex: 1;
  text-align: center;
}
.tag-header.no-logo {
  justify-content: center;
}
.tag-header.no-logo .tag-company {
  flex: 0;
}

/* Generic boxes */
.tag-box {
  border: 1px solid #000;
  border-radius: 1mm;
  text-align: center;
  padding: 1.4mm;
  font-weight: 900;
  font-size: 8.6pt;
  letter-spacing: 0.04em;
}

.tag-brand {
  background: #000000;
  color: #ffffff;
}

.tag-item {
  font-size: 9.2pt;
  line-height: 1.18;
}

/* Attribute rows */
.tag-row {
  display: flex;
  gap: 1mm;
}

.tag-row.prices {
  margin-top: -0.2mm;
}

.tag-cell {
  flex: 1;
  border: 1px solid #000;
  border-radius: 1mm;
  padding: 1.1mm;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 0.7mm;
}

.cell-label {
  font-size: 5.6pt;
  font-weight: 800;
  letter-spacing: 0.24mm;
  color: #000000;
}

.cell-value {
  font-size: 8.6pt;
  font-weight: 900;
  line-height: 1.12;
}

.tag-cell.dark {
  background: #000000;
  color: #ffffff;
}

.tag-cell.dark .cell-label,
.tag-cell.dark .cell-value {
  color: #ffffff;
}

.price-strike {
  text-decoration: line-through;
  text-decoration-thickness: 0.35mm;
}

/* Barcode block */
.barcode {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding-top: 1.2mm;
  padding-bottom: 0.6mm;
  gap: 0.8mm;
}

.barcode-svg {
  width: 42mm;
  height: 15mm;
}

.barcode-text {
  font-size: 7.2pt;
  font-weight: 900;
  letter-spacing: 0.06em;
  color: #000000;
}

.no-barcode {
  font-size: 7.2pt;
  font-weight: 800;
  opacity: 0.55;
  color: #000000;
}

/* Footer */
.tag-footer {
  border-top: 1px dashed #000;
  padding: 1.2mm 1.2mm 1mm;
  margin-top: 1mm;
  background: #000000;
  color: #ffffff;
  font-size: 7.4pt;
  letter-spacing: 0.06em;
  font-weight: 900;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}

@media print {
  .tag {
    page-break-inside: avoid;
  }
  * {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }
}
```

### 3.3. Setting Up and Printing Labels

Follow these steps to configure and use the "Motitaka Product Label" print format:

1.  **Navigate to Print Format List:**
    *   In ERPNext, search for `Print Format` in the awesome bar and open the list.
2.  **Create a New Print Format:**
    *   Click `New`.
    *   **Name:** `Motitaka Product Label`
    *   **DocType:** Select **Item** (This is crucial for the label to appear on product items).
    *   **Module:** `Stock`
    *   Uncheck "Standard".
    *   Check **"Custom Format"**.
3.  **Paste Code:**
    *   In the **"HTML"** field, paste the HTML content from section `3.2. HTML Content`.
    *   In the **"Custom CSS"** field, paste the CSS content from section `3.2. CSS for Label`.
4.  **Save** the Print Format.

**To Print a Label:**

1.  Go to the **Item List** and open the specific item for which you want to print a label.
2.  Click the **"Print" icon** (printer symbol).
3.  Select the **"Motitaka Product Label"** format from the dropdown.
4.  In your browser's print preview dialog, configure the following **crucial settings**:
    *   **Printer:** Select your thermal label printer (e.g., "Zebra ZD420").
    *   **Paper Size:** **This is critical.** You must create or select a custom paper size that matches your label dimensions: **100mm width x 50mm height**.
    *   **Margins:** Set to **"None"** or **"Minimum"** to ensure the design fits perfectly.
    *   **Scale:** Set to **100%** or "Default". Avoid "Fit to Page" as it will distort the design.
    *   **Headers and Footers:** Ensure these are **unchecked** (usually found under "More settings").
5.  **Print.** A precisely sized, high-contrast label should be generated for your product.
