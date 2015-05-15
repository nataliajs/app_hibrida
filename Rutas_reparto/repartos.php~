<?php
/**
 * Desarrollo Web en Entorno Servidor
 * Tema 8 : Aplicaciones web híbridas
 * Ejemplo Rutas de reparto: repartos.php
 */

session_start();

// Incluimos la API de Google
require_once 'libs/google-api-php-client/src/Google/autoload.php';
//require_once '../../libs/google-api-php-client/src/contrib/apiTasksService.php';

// y la librería Xajax
require_once("libs/xajax_core/xajax.inc.php");

$error="";
// Creamos el objeto xajax
$xajax = new xajax('ajaxmaps.php');

// Configuramos la ruta en que se encuentra la carpeta xajax_js
$xajax->configure('javascript URI','./');

// Y registramos las funciones que vamos a llamar desde JavaScript
$xajax->register(XAJAX_FUNCTION,"obtenerCoordenadas");
$xajax->register(XAJAX_FUNCTION,"ordenarReparto");

// Creamos el objeto de la API de Google
//$cliente = new apiClient();
$cliente = new Google_Client();

//mis identificadores
$client_id='1018306128249-2rdh6blf5vfip9957g5mu2p5ej8r8qsc.apps.googleusercontent.com';
$client_secret = 'PnnhEaVwPcnkNQYzzQIvwBWf';
$urlRedirect = 'http://localhost/Rutas_reparto/repartos.php';
$developerkey = 'AIzaSyB5_cbPOao7JmO7rzFlhKOw0NTmJGNeFdo';

// Y lo configuramos con los nuestros identificadores
$cliente->setClientId($client_id);
$cliente->setClientSecret($client_secret);
$cliente->setRedirectUri($urlRedirect);
$cliente->setDeveloperKey($developerkey);

//Establecemos los permisos que queremos otorgar. En este caso queremos conceder acceso a tasks y a calendar para que el usuario pueda acceder a tareas y 
$cliente->setScopes(array('https://www.googleapis.com/auth/tasks','https://www.googleapis.com/auth/calendar'));

// Creamos también un objeto para manejar las listas y sus tareas
//$apitareas = new apiTasksService($cliente);


// Comprobamos o solicitamos la autorización de acceso
/*if (isset($_SESSION['clave_acceso'])) {
  $cliente->setAccessToken($_SESSION['clave_acceso']);
} else {
  $cliente->setAccessToken($cliente->authenticate());
  $_SESSION['clave_acceso'] = $cliente->getAccessToken();
}*/

/************************************************
  If we're logging out we just need to clear our
  local access token in this case
 ************************************************/
if (isset($_REQUEST['logout'])) {
  unset($_SESSION['access_token']);
}
 
/************************************************
  If we have a code back from the OAuth 2.0 flow,
  we need to exchange that with the authenticate()
  function. We store the resultant access token
  bundle in the session, and redirect to ourself.
 ************************************************/
if (isset($_GET['code'])) {
  $cliente->authenticate($_GET['code']);
  $_SESSION['access_token'] = $cliente->getAccessToken();
  header('Location: ' . filter_var($urlRedirect, FILTER_SANITIZE_URL));
}
 
/************************************************
  If we have an access token, we can make
  requests, else we generate an authentication URL.
 ************************************************/
if (isset($_SESSION['access_token']) && $_SESSION['access_token']) {
    $cliente->setAccessToken($_SESSION['access_token']);
        //NATALIA. 'The OAuth 2.0 access token has expired, and a refresh token is not available. 
        //Refresh tokens are not returned for responses that were auto-approved.'
      /*  if($cliente->isAccessTokenExpired()){
            unset($_SESSION['access_token']);
        }else{
            $cliente->setAccessToken($_SESSION['access_token']);
        }*/
} else {
  $authUrl = $cliente->createAuthUrl();
  header('Location: ' . filter_var( $authUrl, FILTER_SANITIZE_URL));
 
}

//Objeto con el api que queremos trabajar en este caso task
$apiTareas= new Google_Service_Tasks($cliente);
 
//Objeto con el api que queremos trabajar con el calendario
$apiCalendario = new Google_Service_Calendar($cliente);

// Comprobamos si se debe ejecutar alguna acción
if (isset($_GET['accion'])) {
    switch ($_GET['accion']) {
        case 'nuevalista':
            if (!empty($_GET['fechaReparto'])) {
                // Crear una nueva lista de reparto
                try {
                    //comprobamos que la fecha es válida
                    $fecha = explode('/' , $_GET['fechaReparto']);
                    if(count($fecha)==3 && checkdate($fecha[1] , $fecha[0] , $fecha[2])){
                        //si es correcta creamos una entrada en Calender e insertamos evento
                        $evento = new Google_Service_Calendar_Event();
                        $evento->setSummary('Reparto');
                        //hora de comienzo
                        $comienzo = new Google_Service_Calendar_EventDateTime();
                        $comienzo->setDateTime("$fecha[2]-$fecha[1]-$fecha[0]T09:00:00.000");
                        $comienzo->setTimeZone("Europe/Madrid");
                        $evento->setStart($comienzo);
                        //hora de terminación
                        $final = new  Google_Service_Calendar_EventDateTime();
                        $final->setDateTime("$fecha[2]-$fecha[1]-$fecha[0]T20:00:00.000");
                        $final->setTimeZone("Europe/Madrid");
                        $evento->setEnd($final);
                        $createdEvent = $apiCalendario->events->insert('primary' , $evento);
                    } else {
                        // La fecha está mal
                        throw new Exception("Fecha incorrecta");
                    }                                         
                    
                    $nuevalista = new Google_Service_Tasks_TaskList();
                    $nuevalista->setTitle('Reparto '.$_GET['fechaReparto']);
                    $apiTareas->tasklists->insert($nuevalista);
                }
                catch (Exception $e) {
                    $error="Se ha producido un error al intentar crear un nuevo reparto.Exception:  ".$e;
                }
            }
            break;
        case 'nuevatarea':
            if (!empty($_GET['nuevotitulo']) && !empty($_GET['idreparto']) && !empty($_GET['latitud']) && !empty($_GET['longitud'])) {
                // Crear una nueva tarea de envío
                try {
                    $nuevatarea = new Google_Service_Tasks_Task();
                    $nuevatarea->setTitle($_GET['nuevotitulo']);
                    if (isset($_GET['direccion'])) 
                        $nuevatarea->setTitle($_GET['nuevotitulo']." - ".$_GET['direccion']);
                    else 
                        $nuevatarea->setTitle($_GET['nuevotitulo']);
                    $nuevatarea->setNotes($_GET['latitud'].",".$_GET['longitud']);
                    // Añadimos la nueva tarea de envío a la lista de reparto
                    $apiTareas->tasks->insert($_GET['idreparto'], $nuevatarea);
                    
                }
                catch (Exception $e) {
                    $error="Se ha producido un error al intentar crear un nuevo envío.";
                }
            }
            break;
        case 'borrarlista':
            if (!empty($_GET['reparto'])) {
                // Borrar una lista de reparto
                try {
                    $reparto = $_GET['reparto'];
                    $apiTareas->tasklists->delete($reparto);
                    $deleteEvent=$apiCalendario->events->delete('primary' , $reparto);
                }
                catch (Exception $e) {
                    $error="Se ha producido un error al intentar borrar el reparto.";
                }
            }
            break;
        case 'borrartarea':
             if (!empty($_GET['reparto']) && !empty($_GET['envio'])) {
                // Borrar una tarea de envío
                try {
                    $apiTareas->tasks->delete($_GET['reparto'],$_GET['envio']);
                }
                catch (Exception $e) {
                    $error="Se ha producido un error al intentar borrar el envío.";
                }                
             } 
             break;
        case 'ordenarEnvios':
             if (!empty($_GET['reparto']) && !empty($_GET['pos'])) {
                // Reordenar las tareas de envío según el orden que se recibe en el array 'pos'
                try {
                    // Primero obtenemos todas las tareas de la lista de reparto
                    $tareas = $apiTareas->tasks->listTasks($_GET['reparto']);
                    
                    // Y después las movemos según la posición recibida en el array 'pos'
                    // El array 'pos' indica la posición que debe tener cada tarea de la lista
                    // $pos[0] = 3 significa que la 1ª tarea (la de índice 0)
                    // debe ponerse en la 4ª posición (la de índice 3)
                    // 
                    // Lo convertimos en el array 'orden' que contiene las tareas que debe haber
                    //  en cada posición de la lista
                    // $orden[3] = 0 significa que en la 4ª posición debemos poner la 1ª tarea 
                    $orden = array_flip($_GET['pos']);
                    
                    // Recorremos el array en orden inverso, esto es, empezando por la tarea
                    //  que debería figurar en última posición de la lista
                    // En cada paso ponemos una tarea en la primera posición de la lista
                    for($i=count($orden)-1; $i>=0; $i--)
                        $apiTareas->tasks->move($_GET['reparto'], $tareas['items'][$orden[$i]]['id']);
                }
                catch (Exception $e) {
                    $error="Se ha producido un error al intentar ordenar los envíos del reparto.";
                }                
             } 
             break;           
    }
}

// Obtenemos el id de la lista de tareas por defecto, para no mostrarla
$listapordefecto = $apiTareas->tasklists->listTasklists();
$id_defecto = $listapordefecto['id'];
?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
    <title>Ejemplo Tema 8: Rutas de reparto</title>
    <link href="estilos.css" rel="stylesheet" type="text/css" />
    <?php
    // Le indicamos a Xajax que incluya el código JavaScript necesario
    $xajax->printJavascript(); 
    ?>    
    <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.6.4/jquery.min.js"></script>
    <script type="text/javascript" src="codigo.js"></script>
</head>

<body>
<div id="dialogo">
    <a id="cerrarDialogo" onclick="ocultarDialogo();">x</a>
    <h1>Datos del nuevo envío</h1>
	<form id="formenvio" name="formenvio" action="<?php echo $_SERVER['PHP_SELF'];?>" method="get">
    	<fieldset>
        <div id="datosDireccion">
            <p>
                <label for='direccion' >Dirección:</label>
                <input type='text' size="30" name='direccion' id='direccion' />
            </p>
            <input type='button' id='obtenerCoordenadas' value='Obtener coordenadas' onclick="getCoordenadas();"/><br />
        </div>
        <div id="datosEnvio">
            <p>
                <label for='latitud' >Latitud:</label>
                <input type='text' size="10" name='latitud' id='latitud' />
            </p>
            <p>
                <label for='longitud' >Longitud:</label>
                <input type='text' size="10" name='longitud' id='longitud' />
            </p>
            <p>
                <label for='altitud' >Altitud:</label>
                <input type='text' size="10" name='altitud' id='altitud' />
            </p>
            <p>
                <label for='nuevotitulo' >Título:</label>
                <input type='text' size="30" name='nuevotitulo' id='titulo' />
            </p>
            <input type='hidden' name='accion' value='nuevatarea' />
            <input type='hidden' name='idreparto' id='idrepartoactual' />
            <input type='submit' id='nuevoEnvio' value='Crear nuevo Envío' />
            <a href="#" onclick="abrirMaps();">Ver en Google Maps</a><br />
        </div>
        </fieldset>
    	</form>
</div>
<div id="fondonegro" onclick="ocultarDialogo();"></div>
<div class="contenedor">
  <div class="encabezado">
    <h1>Ejemplo Tema 8: Rutas de reparto</h1>
    <form id="nuevoreparto" action="<?php echo $_SERVER['PHP_SELF'];?>" method="get">
    <fieldset>
        <input type='hidden' name='accion' value='nuevalista' />
        <input type='submit' id='crearnuevotitulo' value='Crear Nueva Lista de Reparto' />
        <label for='nuevotitulo' >Fecha de reparto:</label>
        <input type='text' name='fechaReparto' id='fechaReparto' />
    </fieldset>
    </form>
  </div>
  <div class="contenido">
<?php
    $repartos = $apiTareas->tasklists->listTasklists();
    // Para cada lista de reparto
    foreach ($repartos['items'] as $reparto) {
        // Excluyendo la lista por defecto de Google Tasks
       // if($reparto['id'] == $id_defecto) continue;
         if($reparto['id'] == '@default') continue;
        
        print '<div id="'.$reparto['id'].'">';
        print '<span class="titulo">'.$reparto['title'].'</span>';
        $idreparto = "'".$reparto['id']."'";
        print '<span class="accion">(<a href="#" onclick="ordenarReparto('.$idreparto.');">Ordenar</a>)</span>';
        print '<span class="accion">(<a href="#" onclick="nuevoEnvio('.$idreparto.');">Nuevo Envío</a>)</span>';
        print '<span class="accion">(<a href="'.$_SERVER['PHP_SELF'].'?accion=borrarlista&reparto='.$reparto['id'].'">Borrar</a>)</span>';
        print '<ul>';
        // Cogemos de la lista de reparto las tareas de envío
        $envios = $apiTareas->tasks->listTasks($reparto['id']);
        
        // Por si no hay tareas de envío en la lista
        if (!empty($envios['items']))
            foreach ($envios['items'] as $envio) {
                // Creamos un elemento para cada una de las tareas de envío
                $idenvio = "'".$envio['id']."'";
                print '<li title="'.$envio['notes'].'" id="'.$idenvio.'">'.$envio['title'].' ('.$envio['notes'].')';
                $coordenadas =  "'".$envio['notes']."'";
                print '<span class="accion">  (<a href="#" onclick="abrirMaps('.$coordenadas.');">Ver mapa</a>)</span>';
                print '<span class="accion">  (<a href="'.$_SERVER['PHP_SELF'].'?accion=borrartarea&reparto='.$reparto['id'].'&envio='.$envio['id'].'">Borrar</a>)</span>';
                print '</li>';
            }
        print '</ul>';
        print '</div>';
    }
?>
  </div>
  <div class="pie">
      <?php print $error; ?>
  </div>
</div>
</body>
</html>