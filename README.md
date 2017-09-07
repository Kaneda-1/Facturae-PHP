# Facturae-PHP

## Qué es
Facturae-PHP es una clase escrita puramente en PHP que permite generar facturas siguiendo el formato estructurado de factura electrónica [Facturae](http://www.facturae.gob.es/) e incluso firmarlas con firma electrónica XAdES sin necesidad de ninguna librería o clase adicional.

### Requisitos
 - PHP 5.6 o superior
 - OpenSSL (solo para firmar facturas)

### Características
- [x] Generación de facturas 100% conformes con la [Ley 25/2013 del 27 de diciembre](https://www.boe.es/diario_boe/txt.php?id=BOE-A-2013-13722) listas para enviar a FACe
- [x] Exportación según el [formato Facturae 3.2.1](http://www.facturae.gob.es/formato/Paginas/version-3-2.aspx)
- [x] Firmado de acuerdo a la [política de firma de Facturae 3.1](http://www.facturae.gob.es/formato/Paginas/politicas-firma-electronica.aspx) basada en XAdES

### Funciones previstas
- [ ] Compatibilidad con el formato Facturae 3.2.2
- [ ] Envío de facturas a FACe directamente desde la clase

## Uso de la clase
Facturae-PHP pretende ser una clase extremadamente rápida y sencilla de usar. A continuación se incluyen varios ejemplos sobre su utilización.
Para más información sobre todos los métodos de Facturae-PHP, la clase se encuentra comentada según bloques de código de [phpDocumentor](https://www.phpdoc.org/).

### Ejemplo básico usando Composer

    // Sistema de carga automática de composer.
    require_once __DIR__ . '/../vendor/autoload.php';

    // Importamos las clases usadas
    use josemmo\Facturae\Facturae;
    use josemmo\Facturae\FacturaeParty;
    
    // Creamos la factura
    $fac = new Facturae();
    
    // Asignamos el número EMP2017120003 a la factura
    // Nótese que Facturae debe recibir el lote y el
    // número separados
    $fac->setNumber('EMP201712', '0003');
    
    // Asignamos el 01/12/2017 como fecha de la factura
    $fac->setIssueNumber('2017-12-01');
    
    // Incluimos los datos del vendedor
    $fac->setSeller(new FacturaeParty([
      "taxNumber" => "A00000000",
      "name"      => "Perico el de los Palotes S.A.",
      "address"   => "C/ Falsa, 123",
      "postCode"  => "123456",
      "town"      => "Madrid",
      "province"  => "Madrid"
    ]);
    
    // Incluimos los datos del comprador
    // Con finos demostrativos el comprador será
    // una persona física en vez de una empresa
    $fac->setBuyer(new FacturaeParty([
      "isLegalEntity" => false,       // Importante!
      "taxNumber"     => "00000000A",
      "name"          => "Antonio",
      "firstSurname"  => "García",
      "lastSurname"   => "Pérez",
      "address"       => "Avda. Mayor, 7",
      "postCode"      => "654321",
      "town"          => "Madrid",
      "province"      => "Madrid"
    ]);
    
    // Añadimos los productos a incluir en la factura
    // En este caso, probaremos con tres lámpara por
    // precio unitario de 20,14€ + 21% IVA
    $fac->addItem("Lámpara de pie", 20.14, 3, Facturae::TAX_IVA, 21);
    
    // Ya solo queda firmar la factura ...
    $fac->sign("ruta/hacia/clave_publica.pem",
      "ruta/hacia/clave_privada.pem", "passphrase");
    
    // ... y exportarlo a un archivo
    $fac->export("ruta/de/salida.xsig");

También se podrían usar directamente las clases mediante un `require_once "ruta/hacia/Facturae.php"; ...`.

### Compradores y vendedores
Los compradores y vendedores son representados en Facturae-PHP con la clase `FacturaeParty` y pueden contener los siguientes atributos:

    $empresa = new FacturaeParty([
      "isLegalEntity" => true, // Se asume true si se omite
      "taxNumber"     => "A00000000",
      "name"          => "Perico el de los Palotes S.A.",
      "address"       => "C/ Falsa, 123",
      "postCode"      => "123456",
      "town"          => "Madrid",
      "province"      => "Madrid",
      "countryCode"   => "ESP",  // Se asume España si se omite
      "book"             => "0",  // Libro
      "merchantRegister" => "RG", // Registro Mercantil
      "sheet"            => "1",  // Hoja
      "folio"            => "2",  // Folio
      "section"          => "3",  // Sección
      "volume"           => "4",  // Tomo
      "email"   => "contacto@perico.com"
      "phone"   => "910555444",
      "fax"     => "910555443"
      "website" => "http://www.perico.com/",
      "contactPeople" => "Perico",
      "cnoCnae" => "4791", // Clasif. Nacional de Act. Económicas
      "ineTownCode" => "280796" // Cód. de municipio del INE
    ]);
    
    $personaFisica = new FacturaeParty([
      "isLegalEntity" => false,
      "taxNumber"     => "00000000A",
      "name"          => "Antonio",
      "firstSurname"  => "García",
      "lastSurname"   => "Pérez",
      "address"       => "Avda. Mayor, 7",
      "postCode"      => "654321",
      "town"          => "Madrid",
      "province"      => "Madrid",
      "countryCode"   => "ESP",  // Se asume España si se omite
      "email"   => "antonio@email.com"
      "phone"   => "910777888",
      "fax"     => "910777888"
      "website" => "http://www.antoniogarcia.es/",
      "contactPeople" => "Antonio García",
      "cnoCnae" => "4791", // Clasif. Nacional de Act. Económicas
      "ineTownCode" => "280796" // Cód. de municipio del INE
    ]);

### Centros administrativos
Para poder emitir facturas a organismos públicos a través de FACe es necesario indicar los centros administrativos relacionados con la administración a facturar.
Estos datos pueden obtenerse a través del [directorio de organismos de FACe](https://face.gob.es/es/directorio/administraciones/) y deben ser indicados a Facturae-PHP de la siguiente forma:

    // Tomamos como ejemplo el Ayuntamiento de San Sebastián de los Reyes
    $ayto = new FacturaeParty([
      "taxNumber" => "P2813400E",
      "name"      => "Ayuntamiento de San Sebastián de los Reyes",
      "address"   => "Plaza de la Constitución, 1",
      "postCode"  => "28701",
      "town"      => "San Sebastián de los Reyes",
      "province"  => "Madrid",
      "centres"   => array(
        new FacturaeCentre([
          "role"     => FacturaeCentre::ROLE_GESTOR,
          "code"     => "L01281343",
          "name"     => "Intervención Municipal",
          "address"  => "Plaza de la Constitución, 1",
          "postCode" => "28701",
          "town"     => "San Sebastián de los Reyes",
          "province" => "Madrid"
        ]),
        new FacturaeCentre([
          "role"     => FacturaeCentre::ROLE_TRAMITADOR,
          "code"     => "L01281343",
          "name"     => "Intervención Municipal",
          "address"  => "Plaza de la Constitución, 1",
          "postCode" => "28701",
          "town"     => "San Sebastián de los Reyes",
          "province" => "Madrid"
        ]),
        new FacturaeCentre([
          "role"     => FacturaeCentre::ROLE_CONTABLE,
          "code"     => "L01281343",
          "name"     => "Intervención Municipal",
          "address"  => "Plaza de la Constitución, 1",
          "postCode" => "28701",
          "town"     => "San Sebastián de los Reyes",
          "province" => "Madrid"
        ])
      )
    ]);

### Uso avanzado de líneas de producto
Es posible añadir una descripción además del concepto al añadir un producto al pasar un `array` de dos elementos en el primer parámetro del método:

    // Solo concepto, sin descripción
    $fac->addItem("Lámpara de pie", 20.14, 3, Facturae::TAX_IVA, 21);
    
    // Con descripción incluída
    $fac->addItem(["Lámpara de pie", "Lámpara de madera de nogal con base rectangular y luz halógena"], 20.14, 3, Facturae::TAX_IVA, 21);

También se puede añadir un elemento sin impuestos, aunque solo debe hacerse en aquellos casos en los que se tenga total certeza:

    $fac->addItem("No llevo impuestos", 100, 1);

La forma adecuada de añadir elementos a los que no se les aplica IVA es la siguiente:

    $fac->addItem("Llevo IVA al 0%", 100, 1, Facturae::TAX_IVA, 0);

Nótese que Facturae-PHP no limita este tipo de comportamientos, es responsabilidad del usuario crear la factura de acuerdo a la legislación aplicable.

### Forma de pago y vencimiento
Es posible indicar la forma de pago de una factura. Por ejemplo, en caso de pagarse al contado:

    $fac->setPaymentMethod(Facturae::PAYMENT_CASH);

En caso de transferencia también debe indicarse la cuenta bancaria destinataria:

    $fac->setPaymentMethod(Facturae::PAYMENT_TRANSFER, "ES7620770024003102575766");

Para establecer la fecha de vencimiento del pago:

    $fac->setDueDate("2017-12-31");

Por defecto, si se establece una forma de pago y no se indica la fecha de vencimiento se interpreta la fecha de la factura como tal.

### Firmado de facturas
Aunque es posible exportar las facturas sin firmarlas, es un paso obligatorio para prácticamente cualquier trámite relacionado con la Administración Pública.
Para firmar facturas se necesita un certificado electrónico (generalmente expedido por la FNMT) del que extraer su clave pública y su clave privada.
Los siguientes comandos permiten extraer el certificado (clave pública) y la clave privada de un archivo PFX:

    openssl pkcs12 -in certificado_de_entrada.pfx -clcerts -nokeys -out clave_publica.pem
    openssl pkcs12 -in certificado_de_entrada.pfx -nocerts -out clave_privada.pem

Por defecto, al firmar una factura se utilizan la fecha y hora actuales como sello de tiempo. Si se quiere indicar otro valor, se debe utilizar el siguiente método:

    $fac->setSignTime("2017-01-01T12:34:56+02:00");

### Otros métodos
Es posible establecer el periodo de facturación con el siguiente método:

    $fac->setBillingPeriod("2017-11-01", "2017-11-30");

También es posible obtener un `array` con los totales de la factura:

    $totales = $fac->getTotals();
