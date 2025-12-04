# Practica-Sensores-Android

Autor: Eric Bermúdez Hernández

Email: alu0101517476@ull.edu.es

--- 

# Práctica: Uso de Sensores en Unity y Android (2025-2026)

Este proyecto consiste en una aplicación desarrollada en **Unity 3D** para la asignatura de Interfaces Inteligentes. El objetivo principal es la implementación y gestión de sensores físicos en dispositivos móviles Android, utilizando tanto el **New Input System** como las APIs clásicas para la geolocalización.

## Descripción del Proyecto

La práctica aborda dos tareas fundamentales de interacción multimodal:

1.  **Monitor de Sensores (Dashboard):** Una interfaz gráfica que detecta, habilita y visualiza en tiempo real los datos crudos de 12 sensores distintos del dispositivo móvil.
2.  **Control Físico del Guerrero:** Un minijuego donde se controla la orientación y el movimiento de un personaje 3D utilizando exclusivamente el movimiento físico del teléfono (Acelerómetro y Brújula) y la ubicación real (GPS).

---

## Stack Tecnológico y Configuración

* **Paquetes:** `com.unity.inputsystem` (Input System).
* **Configuración Crítica del Player:**
    * *Active Input Handling:* **"Both"** (Ambos). Necesario para mezclar `UnityEngine.InputSystem` (Acelerómetro/Giroscopio) con `UnityEngine.Input` (LocationService/Compass).
    * *Orientation:* **Landscape Left** (Horizontal Izquierda) para alinear el sistema de coordenadas del acelerómetro con la vista del juego.

---

## Funcionalidades Implementadas

### 1. Sistema de Monitorización (`SensorMonitor.cs`)
Este script actúa como un gestor centralizado que:
* Solicita permisos de **Actividad Física** (`ACTIVITY_RECOGNITION`) en tiempo de ejecución (necesario para Android 10+).
* Verifica la existencia física del sensor antes de intentar leerlo para evitar errores en dispositivos de gama media/baja.
* Muestra el estado "No disponible" si el hardware carece de un sensor específico (ej. Barómetro o Termómetro).

**Sensores monitorizados:**
* Acelerómetro, Giroscopio, Gravedad, Actitud.
* Aceleración Lineal, Campo Magnético.
* Luz, Presión (Barómetro), Proximidad.
* Humedad, Temperatura Ambiente y Contador de Pasos.

### 2. Controlador del Guerrero (`GuerreroController.cs`)
Script aplicado al personaje para permitir navegación en el mundo real:
* **Orientación (Brújula):** Utiliza `Input.compass.trueHeading` con una interpolación `Quaternion.Slerp` para rotar al personaje suavemente hacia el Norte magnético real.
* **Locomoción (Acelerómetro):** Lee la aceleración en el eje Z mediante el *Input System*. Se invierte el valor (`-z`) para que inclinar el móvil hacia adelante provoque el avance del personaje.
* **Geolocalización (GPS):** Una corrutina inicia el servicio `Input.location`. Si el usuario sale de un radio de coordenadas predefinido (latitud/longitud), el movimiento del personaje se bloquea.

---

## Guía de Desarrollo y Solución de Problemas

Durante el desarrollo de la práctica, se siguieron los siguientes pasos técnicos para asegurar la estabilidad y compatibilidad de la aplicación:

### 1: Configuración del Input System
Se instaló el paquete *Input System* y se configuró el backend en modo **"Both"**. Esto fue crucial porque:
- El *New Input System* maneja mejor los sensores rápidos (Acelerómetro, Giroscopio).

- El *Legacy Input Manager* sigue siendo necesario para acceder al `LocationService` (GPS) y la `Compass` (Brújula), que aún no tienen una implementación estable en el nuevo sistema.

### 2: Gestión de Hardware Inexistente
Inicialmente, la interfaz mostraba "New Text" en sensores que el móvil no tenía. Por tanto, para solucionar esto, se implementó un patrón de verificación de nulos: `if (Sensor.current != null)`. Con esto, ahora la UI informa explícitamente si un sensor no es soportado por el dispositivo o si está desactivado.

### 3: Corrección del Contador de Pasos
El sensor `StepCounter` devolvía siempre valor 0 a pesar de agitar el dispositivo. Esto es debido a que, Android considera los pasos como información sensible y requiere permisos especiales. Para solucionar esto, se hizo lo siguiente:
    1.  Se añadió `<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION" />` al manifiesto.
    2.  Se inyectó código C# (`UnityEngine.Android.Permission`) en el `Start()` para solicitar al usuario permiso mediante una ventana emergente (pop-up).

---

A continuación se muestran tanto los códigos empleados para esta práctica:

### 1. `SensorMonitor.cs` 
```C#
using UnityEngine;
using UnityEngine.InputSystem;
using TMPro;
using UnityEngine.Android; // Necesario para permisos

// Solución al conflicto de nombres del Giroscopio
using Gyroscope = UnityEngine.InputSystem.Gyroscope;

public class SensorMonitor : MonoBehaviour
{
    [Header("Referencias de UI")]
    public TMP_Text accelText;
    public TMP_Text gyroText;
    public TMP_Text magneticText;
    public TMP_Text lightText;
    public TMP_Text pressureText;
    public TMP_Text proximityText;
    public TMP_Text stepText;
    
    [Header("Nuevos Sensores")]
    public TMP_Text gravityText;
    public TMP_Text attitudeText;
    public TMP_Text linearAccelText;
    public TMP_Text humidityText;
    public TMP_Text tempText;

    void Start()
    {
        // 1. Pedir permiso para contar pasos (Solo en Android)
        if (Application.platform == RuntimePlatform.Android)
        {
            if (!Permission.HasUserAuthorizedPermission("android.permission.ACTIVITY_RECOGNITION"))
            {
                Permission.RequestUserPermission("android.permission.ACTIVITY_RECOGNITION");
            }
        }

        // 2. Habilitar todos los sensores (si existen)
        HabilitarSensor(Accelerometer.current);
        HabilitarSensor(Gyroscope.current);
        HabilitarSensor(MagneticFieldSensor.current);
        HabilitarSensor(LightSensor.current);
        HabilitarSensor(PressureSensor.current);
        HabilitarSensor(ProximitySensor.current);
        HabilitarSensor(StepCounter.current);
        HabilitarSensor(GravitySensor.current);
        HabilitarSensor(AttitudeSensor.current);
        HabilitarSensor(LinearAccelerationSensor.current);
        HabilitarSensor(HumiditySensor.current);
        HabilitarSensor(AmbientTemperatureSensor.current);
    }

    // Helper para habilitar código más limpio
    void HabilitarSensor(InputDevice sensor)
    {
        if (sensor != null) InputSystem.EnableDevice(sensor);
    }

    // Helper para deshabilitar (ESTO SUSTITUYE A LA LÍNEA QUE DABA ERROR)
    void DeshabilitarSensor(InputDevice sensor)
    {
        if (sensor != null) InputSystem.DisableDevice(sensor);
    }

    void Update()
    {
        // --- GRUPO 1: Sensores Comunes ---
        ActualizarTexto(accelText, Accelerometer.current, v => ((Accelerometer)v).acceleration.ReadValue().ToString());
        ActualizarTexto(gyroText, Gyroscope.current, v => ((Gyroscope)v).angularVelocity.ReadValue().ToString());
        ActualizarTexto(magneticText, MagneticFieldSensor.current, v => ((MagneticFieldSensor)v).magneticField.ReadValue().ToString());
        ActualizarTexto(lightText, LightSensor.current, v => ((LightSensor)v).lightLevel.ReadValue().ToString() + " lux");
        ActualizarTexto(pressureText, PressureSensor.current, v => ((PressureSensor)v).atmosphericPressure.ReadValue().ToString() + " hPa");
        ActualizarTexto(proximityText, ProximitySensor.current, v => ((ProximitySensor)v).distance.ReadValue().ToString() + " cm");

        // --- GRUPO 2: Contador de Pasos ---
        if (StepCounter.current != null && StepCounter.current.enabled)
        {
            if (Permission.HasUserAuthorizedPermission("android.permission.ACTIVITY_RECOGNITION"))
                 stepText.text = "Steps: " + StepCounter.current.stepCounter.ReadValue().ToString();
            else
            {
                 stepText.text = "Steps: Faltan Permisos";
                 stepText.color = Color.red;
            }
        }
        else
        {
             stepText.text = "Steps: No Soportado";
        }

        // --- GRUPO 3: Sensores Avanzados ---
        ActualizarTexto(gravityText, GravitySensor.current, v => ((GravitySensor)v).gravity.ReadValue().ToString());
        ActualizarTexto(attitudeText, AttitudeSensor.current, v => ((AttitudeSensor)v).attitude.ReadValue().ToString());
        ActualizarTexto(linearAccelText, LinearAccelerationSensor.current, v => ((LinearAccelerationSensor)v).acceleration.ReadValue().ToString());
        ActualizarTexto(humidityText, HumiditySensor.current, v => ((HumiditySensor)v).relativeHumidity.ReadValue().ToString() + " %");
        ActualizarTexto(tempText, AmbientTemperatureSensor.current, v => ((AmbientTemperatureSensor)v).ambientTemperature.ReadValue().ToString() + " C");
    }

    // Helper para actualizar el texto y evitar errores si es null
    void ActualizarTexto(TMP_Text textoUI, InputDevice sensor, System.Func<InputDevice, string> lecturaDatos)
    {
        if (textoUI == null) return; 

        if (sensor != null && sensor.enabled)
        {
            textoUI.text = lecturaDatos(sensor);
            textoUI.color = Color.white;
        }
        else
        {
            textoUI.text = "No disponible";
            textoUI.color = Color.yellow; 
        }
    }

    void OnDisable()
    {
        // --- AQUÍ ESTABA EL ERROR ---
        // Lo hemos cambiado por la desactivación manual uno a uno
        DeshabilitarSensor(Accelerometer.current);
        DeshabilitarSensor(Gyroscope.current);
        DeshabilitarSensor(MagneticFieldSensor.current);
        DeshabilitarSensor(LightSensor.current);
        DeshabilitarSensor(PressureSensor.current);
        DeshabilitarSensor(ProximitySensor.current);
        DeshabilitarSensor(StepCounter.current);
        DeshabilitarSensor(GravitySensor.current);
        DeshabilitarSensor(AttitudeSensor.current);
        DeshabilitarSensor(LinearAccelerationSensor.current);
        DeshabilitarSensor(HumiditySensor.current);
        DeshabilitarSensor(AmbientTemperatureSensor.current);
    }
}

```

### 2. `GuerreroController`

```C#
using UnityEngine;
using UnityEngine.InputSystem; // Necesario para el nuevo sistema
using System.Collections;      // Necesario para las Corrutinas (IEnumerator)

public class GuerreroController : MonoBehaviour
{
    // --- VARIABLES DE CONFIGURACIÓN ---
    [Header("Configuración GPS (Zona Permitida)")]
    // Coordenadas de ejemplo (puedes cambiarlas por las de tu ubicación actual)
    public float centroLatitud = 28.482f;  
    public float centroLongitud = -16.322f; 
    public float radioPermitido = 0.002f; // Rango de movimiento

    [Header("Ajustes de Movimiento")]
    public float velocidadBase = 10.0f;
    public float suavizadoRotacion = 2.0f;

    // --- VARIABLES INTERNAS ---
    private bool gpsListo = false;
    private bool dentroDeRango = true;

    void Start()
    {
        // 1. Iniciamos el servicio de ubicación (GPS)
        StartCoroutine(IniciarServicioGPS());

        // 2. Activamos la Brújula (Compass)
        Input.compass.enabled = true;

        // 3. Activamos el Acelerómetro (Input System)
        if (Accelerometer.current != null)
        {
            InputSystem.EnableDevice(Accelerometer.current);
        }
    }

    // Corrutina para encender el GPS de forma segura
    IEnumerator IniciarServicioGPS()
    {
        // Verificar si el usuario tiene la ubicación activada en el móvil
        if (!Input.location.isEnabledByUser)
        {
            Debug.Log("Permiso de ubicación denegado por el usuario.");
            yield break;
        }

        // Iniciar el servicio con precisión (metros, distancia de actualización)
        Input.location.Start(5f, 5f);

        // Esperar hasta que se inicie (máximo 20 segundos)
        int maxWait = 20;
        while (Input.location.status == LocationServiceStatus.Initializing && maxWait > 0)
        {
            yield return new WaitForSeconds(1);
            maxWait--;
        }

        // Si tardó mucho o falló
        if (maxWait < 1 || Input.location.status == LocationServiceStatus.Failed)
        {
            Debug.Log("No se pudo iniciar el GPS.");
            yield break;
        }

        // ¡GPS Activo!
        gpsListo = true;
    }

    void Update()
    {
        // --- PARTE 1: COMPROBACIÓN GPS ---
        if (gpsListo && Input.location.status == LocationServiceStatus.Running)
        {
            float latitudActual = Input.location.lastData.latitude;
            float longitudActual = Input.location.lastData.longitude;

            // Calculamos la distancia simple (Pitágoras) respecto al centro
            // Nota: Para precisión real se usa la fórmula Haversine, pero para la práctica esto basta.
            float distancia = Mathf.Sqrt(
                Mathf.Pow(latitudActual - centroLatitud, 2) + 
                Mathf.Pow(longitudActual - centroLongitud, 2)
            );

            // Si la distancia es mayor al radio, paramos al guerrero
            if (distancia > radioPermitido)
            {
                dentroDeRango = false;
                // Opcional: Imprimir aviso en consola
            }
            else
            {
                dentroDeRango = true;
            }
        }

        // Si estamos fuera de rango, no ejecutamos movimiento ni rotación (RETURN)
        if (!dentroDeRango) return;


        // --- PARTE 2: ORIENTACIÓN AL NORTE (BRÚJULA) ---
        // Obtenemos hacia dónde está el Norte magnético
        float norteHeading = Input.compass.trueHeading;
        
        // Creamos la rotación objetivo (solo en el eje Y)
        Quaternion rotacionObjetivo = Quaternion.Euler(0, norteHeading, 0);

        // Aplicamos la rotación suavemente (Slerp) como pide el guion
        transform.rotation = Quaternion.Slerp(transform.rotation, rotacionObjetivo, Time.deltaTime * suavizadoRotacion);


        // --- PARTE 3: MOVIMIENTO (ACELERÓMETRO) ---
        // Solo si el acelerómetro existe (para evitar errores en PC)
        if (Accelerometer.current != null)
        {
            // Leemos la aceleración
            Vector3 aceleracion = Accelerometer.current.acceleration.ReadValue();

            // El guion dice: "invertir el valor z" porque al inclinar el móvil hacia adelante,
            // la Z suele ser negativa, pero queremos avanzar (positivo).
            float movimientoAdelante = -aceleracion.z;

            // Calculamos el vector de movimiento:
            // "transform.forward" significa "hacia donde mira el guerrero" (que ya está mirando al Norte)
            Vector3 desplazamiento = transform.forward * movimientoAdelante * velocidadBase * Time.deltaTime;

            // Movemos al guerrero
            transform.Translate(desplazamiento, Space.World);
        }
    }
}

```

En el siguiente vídeo se muestra el funcionamiento de la aplicación:

![Video](Img/Sensores%20edit.gif)

Como se puede apreciar, algunos sensores no los reconoce ya que el dispositivo Android con el que se grabó el vídeo no tiene estos sensores. Estos sensores son los de temperatura ambiente, humedad y presión. Todos los demás sensores funcionaban correctamente, incluido el del guerrero que se rota sobre sí mismo para mirar siempre al norte magnético. Para representar el norte, puse el cubo rojo en la posición (0, 1, 10) que es el que se ve en el vídeo.

En el siguiente vídeo se muestra como es la ejecución del juego en un teléfono, el anterior vídeo era una grabación de pantalla de la ejecución del juego.

![Video 2](Img/Sensores%20IRL.gif)