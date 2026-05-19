# LMSGI_UD07_López_Alonso_Marco

## Generación del Informe Dinámico (QWeb XML):
Para esta primera parte del proyecto voy a diseñar y programar la estructura XML de una plantilla de informe de factura personalizada para "WillmanTech S.L." utilizando la sintaxis de QWeb.
Y para esta primera parte voy a tener en cuenta los siguientes requisistos:
1. Sintaxis estructurada: El documento debe estar bien formado e incluir espacios de nombres correctos si fuera necesario.
2. Lógica embebida QWeb: Debe hacer uso de directivas de iteración (t-foreach) para desglosar dinámicamente las líneas de la factura (invoice_line_ids).
3. Lógica condicional: Debe implementar una condición (t-if) para ocultar la columna de "Descuento" en caso de que ninguna de las líneas de detalle de la factura lo contenga.
4. Data Binding: Utilizar la directiva t-field para mostrar de forma limpia campos como el número de factura (doc.name), fecha de emisión (doc.date) y el total neto (doc.amount_total).

<?xml version="1.0" encoding="UTF-8"?>
<t xmlns:t="http://www.odoo.com/web/xsl" t-name="willmantech.report_invoice_document">
    <!-- Plantilla de Reporte de Factura Personalizada para WillmanTech S.L. -->
    <!-- Requisitos implementados:
        1. Sintaxis estructurada: documento bien formado con espacios de nombres correctos
        2. Lógica embebida QWeb: directivas t-foreach para desglosar líneas de factura (invoice_line_ids)
        3. Lógica condicional: t-if para ocultar columna de descuento si no hay descuentos
        4. Data Binding: t-field para mostrar número (name), fecha (invoice_date) y total neto (amount_total)
    -->
    
    <t t-call="web.external_layout">
        
        <!-- ========== VARIABLES Y CONTEXTO ========== -->
        <t t-set="o" t-value="o.with_context(lang=lang)"/>
        <t t-set="forced_vat" t-value="o.fiscal_position_id.foreign_vat"/>
        <!-- LÓGICA CONDICIONAL: Verificar si existe descuento en alguna línea de producto -->
        <t t-set="has_discount" t-value="any(line.discount for line in o.invoice_line_ids if line.display_type == 'product')"/>
        
        <!-- ========== ENCABEZADO CON DIRECCIÓN DEL CLIENTE ========== -->
        <div class="willmantech-invoice-header">
            <div class="row mb-4">
                <!-- Dirección de Envío (si es diferente de facturación) -->
                <t t-if="o.partner_shipping_id and o.partner_shipping_id != o.partner_id">
                    <div class="col-6">
                        <strong class="d-block mb-2">Dirección de Envío</strong>
                        <div t-field="o.partner_shipping_id" 
                             t-options="{'widget': 'contact', 'fields': ['address', 'name'], 'no_marker': True}"/>
                    </div>
                </t>
                
                <!-- Dirección de Facturación -->
                <div t-attf-class="col-#{6 if (o.partner_shipping_id and o.partner_shipping_id != o.partner_id) else 12}">
                    <strong class="d-block mb-2">Cliente</strong>
                    <address t-field="o.partner_id" 
                             t-options="{'widget': 'contact', 'fields': ['address', 'name'], 'no_marker': True}"/>
                    <t t-if="o.partner_id.vat">
                        <div class="mt-2">
                            <strong>NIF/VAT:</strong> <span t-field="o.partner_id.vat"/>
                        </div>
                    </t>
                </div>
            </div>
        </div>
        
        <!-- ========== CONTENEDOR PRINCIPAL DE LA FACTURA ========== -->
        <div class="willmantech-invoice-container">
            <div class="page">
                
                <!-- ========== INFORMACIÓN GENERAL DE LA FACTURA ========== -->
                <div class="willmantech-invoice-info row mb-4 pb-3 border-bottom">
                    <div class="col-8">
                        <h2 class="mb-3 text-uppercase">
                            <span t-if="o.move_type == 'out_invoice'">FACTURA</span>
                            <span t-elif="o.move_type == 'out_refund'">NOTA DE CRÉDITO</span>
                            <span t-else="">DOCUMENTO</span>
                        </h2>
                        
                        <div class="invoice-details">
                            <!-- DATA BINDING: Mostrar número de factura (doc.name) -->
                            <div class="detail-row mb-2">
                                <span class="detail-label fw-bold">Nº Factura:</span>
                                <strong t-field="o.name" class="ms-2">INV/2023/0001</strong>
                            </div>
                            <!-- DATA BINDING: Mostrar fecha de emisión (doc.date / doc.invoice_date) -->
                            <div class="detail-row mb-2">
                                <span class="detail-label fw-bold">Fecha de Emisión:</span>
                                <strong t-field="o.invoice_date" class="ms-2">2023-09-12</strong>
                            </div>
                            <t t-if="o.invoice_date_due and o.move_type == 'out_invoice'">
                                <div class="detail-row mb-2">
                                    <span class="detail-label fw-bold">Fecha de Vencimiento:</span>
                                    <strong t-field="o.invoice_date_due" class="ms-2">2023-10-31</strong>
                                </div>
                            </t>
                        </div>
                    </div>
                    
                    <!-- Branding WillmanTech S.L. -->
                    <div class="col-4">
                        <div class="company-brand text-end">
                            <p class="company-name mb-0">
                                <strong class="text-uppercase" style="font-size: 1.3rem;">WillmanTech S.L.</strong>
                            </p>
                            <p class="company-info small text-muted" t-field="o.company_id.name">Empresa</p>
                        </div>
                    </div>
                </div>
                
                <!-- ========== TABLA DE LÍNEAS DE FACTURA CON LÓGICA QWEB ========== -->
                <div class="willmantech-invoice-lines mt-4">
                    <table class="table table-bordered table-hover table-sm">
                        <!-- ENCABEZADO DE LA TABLA -->
                        <thead class="table-light">
                            <tr class="fw-bold">
                                <th style="width: 45%;" class="text-start">Descripción del Producto/Servicio</th>
                                <th style="width: 10%;" class="text-center">Cantidad</th>
                                <th style="width: 12%;" class="text-end">Precio Unitario</th>
                                <!-- DIRECTIVA CONDICIONAL: Columna de Descuento - Solo si has_discount == True -->
                                <th t-if="has_discount" style="width: 10%;" class="text-end">Descuento %</th>
                                <th style="width: 15%;" class="text-end">Total</th>
                            </tr>
                        </thead>
                        
                        <!-- CUERPO DE LA TABLA: ITERACIÓN CON t-foreach -->
                        <tbody>
                            <!-- DIRECTIVA t-foreach: Iterar dinámicamente sobre invoice_line_ids -->
                            <t t-foreach="o.invoice_line_ids" t-as="line">
                                
                                <!-- Líneas de PRODUCTO (display_type == 'product') -->
                                <t t-if="line.display_type == 'product'">
                                    <tr class="invoice-line-product">
                                        <!-- Descripción del producto -->
                                        <td class="text-start align-middle">
                                            <span class="fw-bold" t-field="line.product_id.name">Producto XYZ</span>
                                            <t t-if="line.name and len(line.name) > 0">
                                                <div class="text-muted small mt-1" t-out="line.name">Descripción adicional del producto</div>
                                            </t>
                                        </td>
                                        
                                        <!-- Cantidad -->
                                        <td class="text-center align-middle">
                                            <span t-field="line.quantity" t-options="{'widget': 'float'}">1.00</span>
                                            <span class="small ms-1" t-field="line.product_uom_id.name">unidad</span>
                                        </td>
                                        
                                        <!-- Precio Unitario (DATA BINDING con t-field) -->
                                        <td class="text-end align-middle">
                                            <span t-field="line.price_unit" 
                                                  t-options="{'widget': 'monetary', 'display_currency': o.currency_id}">0.00</span>
                                        </td>
                                        
                                        <!-- Descuento (Solo si has_discount es True - DIRECTIVA CONDICIONAL) -->
                                        <td t-if="has_discount" class="text-end align-middle">
                                            <t t-if="line.discount">
                                                <span class="fw-bold" t-out="line.discount" t-options="{'widget': 'float'}">0.00</span>%
                                            </t>
                                            <t t-else="">
                                                <span class="text-muted">-</span>
                                            </t>
                                        </td>
                                        
                                        <!-- Total de línea (DATA BINDING para subtotal) -->
                                        <td class="text-end align-middle fw-bold">
                                            <span t-field="line.price_subtotal" 
                                                  t-options="{'widget': 'monetary', 'display_currency': o.currency_id}">0.00</span>
                                        </td>
                                    </tr>
                                </t>
                                
                                <!-- Líneas de SECCIÓN (display_type == 'line_section') -->
                                <t t-elif="line.display_type == 'line_section'">
                                    <tr class="invoice-line-section table-secondary">
                                        <td colspan="5" class="fw-bold text-uppercase">
                                            <span t-field="line.name">Sección/Grupo de Productos</span>
                                        </td>
                                    </tr>
                                </t>
                                
                                <!-- Líneas de NOTA (display_type == 'line_note') -->
                                <t t-elif="line.display_type == 'line_note'">
                                    <tr class="invoice-line-note table-light">
                                        <td colspan="5" class="text-muted fst-italic">
                                            <span t-field="line.name">Nota o comentario adicional</span>
                                        </td>
                                    </tr>
                                </t>
                                
                            </t>
                        </tbody>
                    </table>
                </div>
                
                <!-- ========== SECCIÓN DE TOTALES ========== -->
                <div class="willmantech-totals mt-4">
                    <div class="row">
                        <div class="col-7"></div>
                        <div class="col-5">
                            <table class="table table-borderless table-sm">
                                <tbody>
                                    <!-- Subtotal (Sin Impuestos) -->
                                    <tr>
                                        <td class="text-end"><strong>Subtotal:</strong></td>
                                        <td class="text-end fw-bold" style="width: 40%;">
                                            <span t-field="o.amount_untaxed" 
                                                  t-options="{'widget': 'monetary', 'display_currency': o.currency_id}">0.00</span>
                                        </td>
                                    </tr>
                                    
                                    <!-- Impuestos (si existen) -->
                                    <t t-if="o.amount_tax > 0">
                                        <tr class="border-top">
                                            <td class="text-end"><strong>Impuestos:</strong></td>
                                            <td class="text-end">
                                                <span t-field="o.amount_tax" 
                                                      t-options="{'widget': 'monetary', 'display_currency': o.currency_id}">0.00</span>
                                            </td>
                                        </tr>
                                    </t>
                                    
                                    <!-- TOTAL NETO A PAGAR (DATA BINDING con t-field para amount_total) -->
                                    <tr class="table-active fw-bold border-top-3 border-bottom-3">
                                        <td class="text-end text-uppercase">
                                            <strong>Total Neto a Pagar:</strong>
                                        </td>
                                        <td class="text-end" style="font-size: 1.1rem;">
                                            <span t-field="o.amount_total" 
                                                  t-options="{'widget': 'monetary', 'display_currency': o.currency_id}">0.00</span>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
                
                <!-- ========== CONDICIONES DE PAGO ========== -->
                <div class="willmantech-payment-terms mt-4 pt-3 border-top">
                    <h6 class="fw-bold mb-2">Condiciones de Pago</h6>
                    <t t-if="o.invoice_payment_term_id">
                        <p class="small text-muted">
                            <span t-field="o.invoice_payment_term_id.note">Condiciones de pago estándar</span>
                        </p>
                    </t>
                    <t t-if="o.payment_reference">
                        <p class="small">
                            <strong>Referencia de Pago:</strong> 
                            <span class="fw-bold" t-field="o.payment_reference">REF-001</span>
                        </p>
                    </t>
                </div>
                
                <!-- ========== NOTAS Y OBSERVACIONES ========== -->
                <t t-if="o.narration and not is_html_empty(o.narration)">
                    <div class="willmantech-notes mt-4 pt-3 border-top">
                        <h6 class="fw-bold mb-2">Notas y Observaciones</h6>
                        <div class="small text-muted" t-field="o.narration">
                            Notas adicionales sobre la factura
                        </div>
                    </div>
                </t>
                
            </div>
        </div>
        
    </t>
    
</t>
