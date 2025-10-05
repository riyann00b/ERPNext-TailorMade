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
<!--
    REDESIGNED PROFESSIONAL THERMAL LABEL
    50mm x 100mm with Modern UI/UX Design
    Optimized spacing, visual hierarchy, and readability
-->
<div class="tag-container">
  
  <!-- Brand Header Section -->
  <div class="brand-header">
    {% set logo_file = frappe.get_all('File',
      filters={
        'attached_to_doctype': 'Company',
        'attached_to_name': frappe.db.get_value("Global Defaults", None, "default_company")
      },
      fields=['file_url'], limit=1) %}
    
    <div class="logo-section">
      {% if logo_file %}
        <img src="{{ logo_file.file_url }}" class="company-logo" alt="Logo">
      {% endif %}
      <div class="company-info">
        <div class="company-name">{{ frappe.db.get_value("Global Defaults", None, "default_company") or "MOTI TAKA" }}</div>
        {% if doc.brand %}
        <div class="brand-badge">{{ doc.brand }}</div>
        {% endif %}
      </div>
    </div>
  </div>
  
  <!-- Product Identity Section -->
  <div class="product-section">
    <div class="item-name">{{ doc.item_name or "PRODUCT NAME" }}</div>
  </div>
  
  <!-- Product Details Grid -->
  <div class="details-grid">
    <div class="detail-item">
      <span class="detail-label">COLOR</span>
      <span class="detail-value">{{ doc.custom_color or "N/A" }}</span>
    </div>
    
    {% set ig = (doc.item_group or '')|lower %}
    {% set pc = (doc.custom_product_category or '')|lower %}
    {% if not ('dress material' == ig or 'dress material' == pc) %}
    <div class="detail-item">
      <span class="detail-label">SIZE</span>
      <span class="detail-value">{{ doc.custom_size or "N/A" }}</span>
    </div>
    {% endif %}
  </div>
  
  <!-- Pricing Section -->
  <div class="pricing-section">
    {% set std = doc.standard_rate or 0 %}
    {% set offer = (std / 2) %}
    
    <div class="price-container">
      <div class="mrp-section">
        <span class="price-label">M.R.P</span>
        <span class="mrp-price">{{ frappe.format(std, df={"fieldtype":"Currency","options":doc.currency}) }}</span>
      </div>
      <div class="offer-section">
        <span class="price-label">OFFER</span>
        <span class="offer-price">{{ frappe.format(offer, df={"fieldtype":"Currency","options":doc.currency}) }}</span>
      </div>
    </div>
  </div>
  
  <!-- Barcode Section -->
  <div class="barcode-section">
    {% set code = (doc.barcodes and doc.barcodes and doc.barcodes.barcode) and doc.barcodes.barcode or '' %}
    {% if code %}
      <div class="barcode-wrapper">
        <canvas id="barcode-1" class="barcode-canvas"></canvas>
      </div>
    {% else %}
      <div class="no-barcode-display">
        <div class="no-barcode-text">NO BARCODE</div>
      </div>
    {% endif %}
  </div>
  
  <!-- Footer Section -->
  <div class="footer-section">
    <div class="internal-reference">{{ (doc.custom_encrypted_buying_price or doc.item_code or '')|upper }}</div>
  </div>
  
</div>

<script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.12.1/dist/JsBarcode.all.min.js"></script>

<script>
(function(){
  try {
    if (typeof JsBarcode === 'undefined') return;
    var code = "{{ code }}".trim();
    var canvas = document.getElementById('barcode-1');
    if (!canvas || !code) return;
    JsBarcode(canvas, code, {
      format: "CODE128",
      width: 2,
      height: 50,
      displayValue: false,
      margin: 8,
      background: "#FFFFFF",
      lineColor: "#000000"
    });
  } catch(e){ console.error(e); }
})();
</script>
```

**CSS for Label:**
```css
/*
    REDESIGNED PROFESSIONAL THERMAL LABEL
    50mm x 100mm with Modern UI/UX Design
    Optimized spacing, visual hierarchy, and readability
*/
.tag-container {
  width: 100mm; /* Adjust container width to match label width */
  font-family: 'SF Mono', 'Courier New', monospace;
  font-size: 12px; /* Slightly smaller font for labels */
  color: #000;
  font-weight: bold;
  text-align: center;
  padding: 2mm; /* Add padding for better spacing */
  box-sizing: border-box;
}

/* Brand Header */
.brand-header {
  margin-bottom: 4px;
}
.logo-section {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 3px;
}
.company-logo {
  max-width: 40px; /* Smaller logo for labels */
  max-height: 40px;
  margin-right: 5px;
}
.company-info {
  line-height: 1.3;
}
.company-name {
  font-size: 16px;
  font-weight: bold;
  margin: 0;
}
.brand-badge {
  font-size: 10px;
  font-weight: normal;
  color: #555;
}

/* Product Identity */
.product-section {
  margin-bottom: 6px;
}
.item-name {
  font-size: 16px;
  font-weight: bold;
  word-wrap: break-word; /* Ensure long names wrap */
}

/* Product Details Grid */
.details-grid {
  display: flex;
  justify-content: space-around;
  margin-bottom: 6px;
}
.detail-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  line-height: 1.2;
}
.detail-label {
  font-size: 10px;
  font-weight: normal;
  color: #666;
  margin-bottom: 1px;
}
.detail-value {
  font-size: 14px;
  font-weight: bold;
}

/* Pricing Section */
.pricing-section {
  margin-bottom: 8px;
}
.price-container {
  display: flex;
  justify-content: space-around;
  align-items: baseline;
}
.mrp-section, .offer-section {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.price-label {
  font-size: 10px;
  font-weight: normal;
  color: #555;
}
.mrp-price, .offer-price {
  font-size: 16px;
  font-weight: bold;
}
.offer-price {
  color: #e60000; /* Highlight offer price */
}

/* Barcode Section */
.barcode-section {
  text-align: center;
  margin-bottom: 5px;
}
.barcode-wrapper {
  display: inline-block;
  padding: 3px;
  background-color: #FFF; /* White background for barcode */
  border: 1px solid #DDD; /* Light border */
}
.barcode-canvas {
  display: block; /* Removes extra space below canvas */
}
.no-barcode-display {
  height: 60px; /* Match barcode height */
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px dashed #CCC;
  background-color: #f9f9f9;
}
.no-barcode-text {
  font-size: 14px;
  font-weight: bold;
  color: #888;
}

/* Footer Section */
.footer-section {
  font-size: 10px;
  font-weight: normal;
  color: #555;
  line-height: 1.3;
  margin-top: 8px;
}
.internal-reference {
  font-weight: bold;
  font-size: 12px;
  word-wrap: break-word;
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
