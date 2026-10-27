# PracticaAppium

Guía introductoria para automatizar aplicaciones móviles con Appium.

## ¿Qué es Appium?

Appium permite controlar una aplicación iOS o Android desde un programa de prueba. El cliente de prueba se conecta al servidor Appium; este usa un controlador de plataforma, como UiAutomator2 para Android, para comunicarse con el dispositivo o emulador.

## Preparar Android

Necesitas Node.js y npm, Python 3, un JDK configurado, el Android SDK (con `ANDROID_HOME` y `adb` disponibles) y un emulador iniciado o un dispositivo conectado con la depuración USB activada. Comprueba que Android detecta el dispositivo:

```sh
adb devices
```

Instala el servidor Appium y el controlador de Android:

```sh
npm install --global appium
appium driver install uiautomator2
```

En una terminal, inicia el servidor:

```sh
appium
```

En otra terminal, instala el cliente de Python:

```sh
python -m pip install Appium-Python-Client
```

## Primera prueba

Guarda este ejemplo como `basic_test.py`. Cambia la ruta del APK y el identificador de accesibilidad para que coincidan con tu aplicación. En Android, el identificador de accesibilidad suele corresponder al atributo `content-desc` del elemento.

```python
import os
from pathlib import Path

from appium import webdriver
from appium.options.android import UiAutomator2Options
from appium.webdriver.common.appiumby import AppiumBy
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait


app_path = Path(os.environ["APP_PATH"]).expanduser().resolve()
if not app_path.is_file():
    raise FileNotFoundError(f"No se encontró el APK: {app_path}")

accessibility_id = os.environ.get("ACCESSIBILITY_ID", "home-title")
capabilities = {
    "platformName": "Android",
    "appium:automationName": "UiAutomator2",
    "appium:deviceName": "Android Emulator",
    "appium:app": str(app_path),
}

driver = webdriver.Remote(
    "http://127.0.0.1:4723",
    options=UiAutomator2Options().load_capabilities(capabilities),
)

try:
    title = WebDriverWait(driver, 10).until(
        EC.visibility_of_element_located(
            (AppiumBy.ACCESSIBILITY_ID, accessibility_id)
        )
    )
    assert title.is_displayed()
    print("La pantalla principal está visible.")
finally:
    driver.quit()
```

Ejecuta la prueba indicando el APK y el identificador del elemento:

```sh
APP_PATH=/ruta/a/mi-app.apk ACCESSIBILITY_ID=home-title python basic_test.py
```

En PowerShell, configura las mismas variables antes de ejecutar el comando:

```powershell
$env:APP_PATH = "C:\ruta\mi-app.apk"
$env:ACCESSIBILITY_ID = "home-title"
python basic_test.py
```

La prueba inicia una sesión, espera hasta diez segundos a que aparezca el elemento y cierra la sesión incluso si falla la comprobación. La aplicación debe incluir el identificador indicado; sustituye `home-title` por uno que exista en tu app.

## Localizar e interactuar con elementos

Prefiere selectores estables, como un identificador de accesibilidad o el identificador de recurso de la aplicación, antes que selectores basados en la posición o en textos que pueden cambiar. Para tocar un botón, espera a que esté disponible y luego llama a `click()`:

```python
button = WebDriverWait(driver, 10).until(
    EC.element_to_be_clickable((AppiumBy.ACCESSIBILITY_ID, "continue-button"))
)
button.click()
```

`AppiumBy` también ofrece estrategias como `ID` y `ANDROID_UIAUTOMATOR`. Mantén las esperas explícitas cerca de la acción que las necesita y cierra siempre la sesión con `driver.quit()`.

## iOS

El ejemplo utiliza UiAutomator2 y solo sirve para Android. Para automatizar iOS necesitas macOS, Xcode y el controlador XCUITest de Appium; configura las capacidades y el cliente de acuerdo con el simulador o dispositivo iOS que vayas a usar.
