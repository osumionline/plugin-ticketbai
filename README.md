Osumi Framework Plugins: `OTicketBai`

Este plugin añade la clase `OTicketBai` al framework con la que se pueden hacer llamadas al servicio [TicketBaiWS](https://ticketbaiws.eus) de [Berein](https://www.berein.com/). La consiguración se realiza en el archivo `Config.json` general de la aplicación. Es necesario estar registrado en TicketBaiWS y obtener el `token` y el `nif` de su panel de configuración.

Configuración

```json
{
  ...,
  "plugins": {
    "ticketbai": {
        "token": "asdf123...",
        "nif": "12345678Z"
    }
  },
}
```

Uso del plugin

```php
$tbai = new OTicketBai(true); // true sirve para indicar producción y false para indicar el entorno test

// Primero se comprueba el estado del servicio y si está activo se pueden realizar las peticiones
if ($tbai->checkStatus()) {
  /**
	 * Este método permite enviar una factura a la hacienda foral correspondiente y devolverá la huella TBAI, la imagen código QR en base64
	 * y la URL de validación de la factura de la hacienda foral que contiene el QR. El entorno de test permite generar TBAIs en el entorno
	 * de pruebas de la hacienda correspondiente.
   */

  $datosTBai = [
    'fecha'                     => date('d/m/Y', time()),
    'hora'                      => date('H:i:s', time()),
    'nif'                       => '',
    'nombre'                    => '',
    'direccion'                 => '',
    'cp'                        => '',
    'serie'                     => 'TPV01',
    'numero'                    => sprintf('%06d', $num_venta),
    'simplificada'              => true,
    'modo_recargo_equivalencia' => true,
    'rectificativa'             => false,
    'importacion'               => false,
    'intracomunitaria'          => false,
    'retencion'                 => 0,
    'lineas'                    => [],
    'total_factura'             => $total_factura
  ];

  // Por cada línea de la venta se crea un objeto datos_linea
  foreach ($lineas as $linea) {
    $importe_siva = $linea->get('pvp') / (1 + ($linea->get('iva') / 100));

    $datos_linea = [
      'iva'              => ($linea->get('iva') == 0) ? 21 : $linea->get('iva'),
      'descripcion'      => html_entity_decode($linea->get('nombre_articulo')),
      'cantidad'         => $linea->get('unidades'),
      'importe_unitario' => round($importe_siva, 4),
      'tipo_iva'         => $linea->get('iva'),
      'tipo_req'         => 0
    ];

    array_push($datosTBai['lineas'], $datos_linea);
  }

  // Se envía el objeto con todos los datos de la factura y las líneas de ventas
  $response = $tbai->nuevoTbai($datosTBai);
  if (is_array($response)) {
    echo "TicketBai response OK";
    // $response['huella_tbai'] Datos de la huella TicketBai
    // $response['qr'] Imagen del código QR en formato Base64
    // $response['url']  URL de validación de la factura de la hacienda foral
  }
  else {
    echo "TicketBai response ERROR";
  }
}
```
