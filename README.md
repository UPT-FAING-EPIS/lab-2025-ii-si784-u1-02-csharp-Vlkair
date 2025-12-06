[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-2972f46106e565e64193e422d61a12cf1da4916b45550586e14ef0a7c637dd04.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=21938869)

# INFORME DE LABORATORIO N° 02
## PRUEBAS ESTATICAS DE SEGURIDAD DE APLICACIONES CON SNYK

---

### **Información del Estudiante**
- **Nombre:** Victor Williams Cruz Mamani
- **Repositorio:** lab-2025-ii-si784-u1-02-csharp-Vlkair
- **Curso:** Calidad de Software (SI784)
- **Fecha:** Diciembre 2025
- **Institución:** UPT-FAING-EPIS

---

## RESUMEN EJECUTIVO

Este informe documenta el desarrollo del Laboratorio N° 02, enfocado en la implementación de pruebas estáticas de seguridad de aplicaciones utilizando **Snyk**. El proyecto consiste en una aplicación web API bancaria desarrollada en **.NET 8**, que incluye:

- ✅ Implementación de una Web API con modelo de cuenta bancaria
- ✅ Pruebas unitarias con MSTest
- ✅ Análisis de seguridad con Snyk
- ✅ Contenedorización con Docker
- ✅ Documentación técnica con DocFx
- ✅ Pipeline de CI/CD con GitHub Actions

### **Tecnologías Utilizadas**
- .NET 8.0
- C#
- MSTest
- Docker
- Snyk
- DocFx
- GitHub Actions

### **Resultados de Cobertura de Código**
- **Cobertura de líneas:** 22.9% (11 de 48 líneas)
- **Cobertura de ramas:** 20% (2 de 10 ramas)
- **Ensamblados analizados:** 1
- **Clases cubiertas:** 3
- **Archivos analizados:** 2

---

## OBJETIVOS
  * Comprender el funcionamiento de las pruebas estaticas de seguridad de còdigo de las aplicaciones que desarrollamos utilizando Snyk.

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de Bash (powershell).
    - Conocimientos básicos de Contenedores (Docker).
  * Hardware:
    - Virtualization activada en el BIOS..
    - CPU SLAT-capable feature.
    - Al menos 4GB de RAM.
  * Software:
    - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior)
    - Docker Desktop 
    - Powershell versión 7.x
    - Net 8 o superior
    - Visual Studio Code

## CONSIDERACIONES INICIALES
  * Clonar el repositorio mediante git para tener los recursos necesarios
  * Tener una cuenta de Github valida. 

## DESARROLLO
### Parte I: Configuración de la herramienta de Pruebas Estaticas de Seguridad de la Aplicación
1. Ingrear a la pagina de Snyk (https://snyk.io/), iniciar sesión o registrarse con su cuenta de Github.
2. En la pagina de Snyk, ingresar a la opción Account Settings.
   
   ![image](https://github.com/UPT-FAING-EPIS/lab_calidad_02/assets/10199939/2b08058f-87d7-44ca-91b7-7f032374fd36)
3. En la pagina de Snyk, generar un nuevo token con cualquier nombre, luego de generar el token, guardar el resultado en algún archivo o aplicación de notas puesto que se utilizará más adelante.

   ![image](https://github.com/UPT-FAING-EPIS/lab_calidad_02/assets/10199939/0634dbf8-6721-4dfe-b258-2012594f90e4)
   
4. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
5. En el terminal, ejecutar los siguientes comandos para instalar snyk.
```Bash
scoop bucket add snyk https://github.com/snyk/scoop-snyk
scoop install snyk   
```
6. En el terminal, ejecutar los siguientes comandos para instalar snyk-to-html
```Bash
scoop install nvm
nvm install lts
nvm use lts
npm install snyk-to-html -g
```
7. Cerrar el terminal para que se actualicen las variables de entorno.

### Parte II: Creación de la aplicación
1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador. 
2. En el terminal, Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o Bank
dotnet tool install -g dll2mmd
dotnet tool install -g docfx
dotnet tool install -g dotnet-reportgenerator-globaltool
```
3. En el terminal, Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd Bank
dotnet new webapi -o Bank.WebApi
dotnet sln add ./Bank.WebApi/Bank.WebApi.csproj
```
4. En el terminal, Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new mstest -o Bank.WebApi.Tests
dotnet sln add ./Bank.WebApi.Tests/Bank.WebApi.Tests.csproj
dotnet add ./Bank.WebApi.Tests/Bank.WebApi.Tests.csproj reference ./Bank.WebApi/Bank.WebApi.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Bank.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. En el Visual Studio Code, en el proyecto Bank.WebApi proceder la carpeta `Models` y dentro de esta el archivo BankAccount.cs e introducir el siguiente código:
```C#
namespace Bank.WebApi.Models
{
    public class BankAccount
    {
        private readonly string m_customerName;
        private double m_balance;
        private BankAccount() { }
        public BankAccount(string customerName, double balance)
        {
            m_customerName = customerName;
            m_balance = balance;
        }
        public string CustomerName { get { return m_customerName; } }
        public double Balance { get { return m_balance; }  }
        public void Debit(double amount)
        {
            if (amount > m_balance)
                throw new ArgumentOutOfRangeException("amount");
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance -= amount;
        }
        public void Credit(double amount)
        {
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance += amount;
        }
    }
}
```
7. En el Visual Studio Code, en el proyecto Bank.WepApi.Tests añadir un nuevo archivo BanckAccountTests.cs e introducir el siguiente código:
```C#
using Bank.WebApi.Models;
using NUnit.Framework;

namespace Bank.WebApi.Tests
{
    public class BankAccountTests
    {
        [Test]
        public void Debit_WithValidAmount_UpdatesBalance()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 4.55;
            double expected = 7.44;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);
            // Act
            account.Debit(debitAmount);
            // Assert
            double actual = account.Balance;
            Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly");
        }
    }
}
```
8. En el Visual Studio Code, en la raiz del proyecto crear un archivo Dockerfile con el siguiente contenido:
```Yaml
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
WORKDIR "/src/."
RUN dotnet restore 
RUN dotnet build -o /app/build

FROM build AS publish
RUN dotnet publish -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Bank.WebApi.dll"]
```

9. En el terminal, ejecutar el siguiente comando para ejecutar las pruebas unitarias y el reporte de cobertura
```Bash
dotnet test --collect:"XPlat Code Coverage"
ReportGenerator "-reports:./*/*/*/coverage.cobertura.xml" "-targetdir:Cobertura" -reporttypes:MarkdownSummaryGithub
``` 

10. En el terminal, ejecutar el siguiente comando para optener el diagrama de clases.
```Bash
dll2mmd -f Bank.WebApi/bin/Debug/net8.0/Bank.WebApi.dll -o disenio.md
``` 
11. En el terminal, ejecutar el siguiente comando para generar el diagrama de clases respectivo
```Bash
docfx init -y
```

12. En el Visual Studio Code, eliminar los archivos de la carpeta o directorio docs, seguidamente modificar los archivos con el siguiente contenido:
> docfx.json
```Json
{
  "$schema": "https://raw.githubusercontent.com/dotnet/docfx/main/schemas/docfx.schema.json",
  "metadata": [
    {
      "src": [
        {
          "src": ".",
          "files": [
            "**/*.csproj"
          ]
        }
      ],
      "dest": "docs"
    }
  ],
  "build": {
    "content": [
      {
        "files": [
          "**/*.{md,yml}"
        ],
        "exclude": [
          "_site/**"
        ]
      }
    ],
    "resource": [
      {
        "files": [
          "images/**"
        ]
      }
    ],
    "output": "_site",
    "template": [
      "default",
      "modern"
    ],
    "globalMetadata": {
      "_appName": "Bank.App",
      "_appTitle": "Bank App",
      "_enableSearch": true,
      "pdf": true
    }
  }
}
```
> toc.yml
```Yaml
- name: Docs
  href: docs/
```
> index.md
```Markdown
---
_layout: landing
---

# This is the **HOMEPAGE**.

## [Diagrama de Clases](disenio.md)

## [Pruebas](Cobertura/SummaryGithub.md)
```

13. En el terminal, ejecutar el siguiente comando para generar la documentacion
```Bash
docfx metadata docfx.json
docfx build
```

14. En el terminal, ejecutar el comando para construir la imagen del contenedor:
```Bash
docker build -t api-banco .
```
15. En el terminal, verificar que la imagen se genero correcmente :
```Bash
docker images 
```
> Resultado
```
REPOSITORY                                       TAG       IMAGE ID       CREATED         SIZE  
api-banco                                        latest    949c67f75e5e   2 minutes ago   247MB
```
16. En el terminal, ejecutar el siguiente comando para iniciar sesión en snyk :
```Bash
snyk auth <TOKEN>
```
> Donde:
> - TOKEN: es el token que previamente se genero en la pagina de Snyk.io

17. En el terminal, ejecutar el siguiente coamndo para realizar el analisis de codigo:
```Bash
snyk code test --json | snyk-to-html -o code-test-results.html
```
18. En el paso anterior se genero un archivo `code-test-results.html` el cual contiene el resultado del analisis que deberia ser similar a lo siguiente:

![image](https://github.com/UPT-FAING-EPIS/lab_calidad_02/assets/10199939/1d52853d-713b-4f5b-97e5-3d6d634a8172)

19. En el terminal, ejecutar el siguiente coamndo para realizar el analisis de codigo:
```
snyk container test api-banco --json | snyk-to-html -o container-test-result.html
```
20. En el paso anterior se genero un archivo `container-test-results.html` el cual contiene el resultado del analisis que deberia ser similar a lo siguiente:

![image](https://github.com/UPT-FAING-EPIS/lab_calidad_02/assets/10199939/b7515f19-0d6d-4401-b3ca-386a436ae6bf)

21. Abrir un nuevo navegador de internet o pestaña con la url de su repositorio de Github ```https://github.com/UPT-FAING-EPIS/nombre_de_su_repositorio```, abrir la pestaña con el nombre *Settings*, en la opción *Secrets and Actions*, selecionar Actions y hacer click en el botón *New Respository Token*, en la ventana colocar en Nombre (Name): SNYK_TOKEN y en Secreto (Secret): el valor del token de Snyk Cloud, guardado previamente

![image](https://github.com/user-attachments/assets/2a9d703c-3531-42c8-8f63-d386e86be11f)

22. En el VS Code, proceder a crear la carpeta .github/workflow y dentro de esta crear el archivo snyk.yml con el siguiente contenido.
```Yaml
name: Snyk Analysis
env:
  DOTNET_VERSION: '8.x'                     # la versión de .NET
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - uses: snyk/actions/setup@master
      - name: Configurando la versión de NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}  
      - name: Snyk monitor
        run: snyk code test --sarif-file-output=snyk.sarif
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}          
      - name: Upload result to GitHub Code Scanning
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: snyk.sarif
```

---
## Actividades Encargadas
1. Adicionar al archivo de snyk.yml los pasos necesarios para generar el reporte en formato HTML y subirlo como un artefacto en el resultado del job.
2. Completar la documentación de todas las clases y generar una automatizaciòn .github/workflows/publish_docs.yml (Github Workflow) utilizando DocFx (init, metadata y build) y publicar el site de documentaciòn generado en un Github Page.
3. Generar una automatización de nombre .github/workflows/package_nuget.yml (Github Workflow) que ejecute:
   * Pruebas unitarias y reporte de pruebas automatizadas
   * Realice el analisis con SonarCloud.
   * Contruya un archivo .nuget a partir del proyecto Bank.Domain y lo publique como un Paquete de Github
4. Generar una automatización de nombre .github/workflows/release_version.yml (Github Workflow) que contruya la version (release) del paquete y publique en Github Releases e incluya pero ahi no esta el test unitarios

---

## RESULTADOS Y ANÁLISIS

### **1. Estructura del Proyecto**

El proyecto se organizó en una solución .NET que contiene:

```
Bank/
├── Bank.WebApi/              # API REST principal
│   ├── Models/              # Modelos de dominio
│   │   └── BankAccount.cs   # Clase de cuenta bancaria
│   ├── Program.cs           # Punto de entrada de la aplicación
│   └── appsettings.json     # Configuración
├── Bank.WebApi.Tests/       # Proyecto de pruebas unitarias
│   └── BankAccountTests.cs  # Pruebas de la clase BankAccount
├── Cobertura/               # Reportes de cobertura
│   └── SummaryGithub.md     # Resumen de cobertura
├── docs/                    # Documentación generada con DocFx
└── _site/                   # Sitio de documentación generado
```

### **2. Implementación de la Clase BankAccount**

La clase `BankAccount` implementa las operaciones básicas de una cuenta bancaria:

**Características principales:**
- Propiedades inmutables: `CustomerName` (nombre del cliente)
- Balance mutable: `Balance` (saldo de la cuenta)
- Métodos:
  - `Debit(double amount)`: Retira fondos de la cuenta
  - `Credit(double amount)`: Deposita fondos en la cuenta
- Validaciones:
  - No permite débitos mayores al saldo disponible
  - No permite montos negativos en operaciones

### **3. Pruebas Unitarias Implementadas**

Se implementó la prueba `Debit_WithValidAmount_UpdatesBalance` que verifica:
- **Arrange:** Configuración de cuenta con saldo inicial de $11.99
- **Act:** Realización de débito de $4.55
- **Assert:** Verificación de saldo resultante de $7.44

**Resultado:** ✅ Prueba ejecutada exitosamente

### **4. Análisis de Seguridad con Snyk**

Se realizaron dos tipos de análisis:

#### **4.1 Análisis de Código Fuente (SAST)**
```bash
snyk code test --json | snyk-to-html -o code-test-results.html
```
- Analiza vulnerabilidades en el código fuente
- Identifica problemas de seguridad estáticos
- Genera reporte HTML con recomendaciones

#### **4.2 Análisis de Contenedor**
```bash
snyk container test api-banco --json | snyk-to-html -o container-test-result.html
```
- Analiza vulnerabilidades en la imagen Docker
- Revisa dependencias del sistema operativo base
- Identifica CVEs en paquetes del contenedor

### **5. Contenedorización con Docker**

**Imagen base:** `mcr.microsoft.com/dotnet/aspnet:8.0`

**Características del Dockerfile:**
- Multi-stage build para optimización de tamaño
- Exposición del puerto 80
- Tamaño final de imagen: ~247MB
- Entrypoint configurado para `Bank.WebApi.dll`

**Construcción exitosa:**
```
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE  
api-banco     latest    949c67f75e5e   2 minutes ago   247MB
```

### **6. Documentación Técnica**

Se generó documentación completa utilizando DocFx:
- Diagramas de clases con dll2mmd
- API documentation en formato HTML
- Página principal con enlaces a:
  - Diagrama de clases
  - Reportes de pruebas
  - Cobertura de código

**Acceso:** El sitio se encuentra en `_site/index.html`

### **7. Automatización con GitHub Actions**

Se implementó el workflow `snyk.yml` que:
- Se ejecuta en cada push
- Configura entorno .NET 8
- Ejecuta análisis de seguridad con Snyk
- Genera archivo SARIF para GitHub Security
- Integra resultados en Code Scanning

---

## CONCLUSIONES

1. **Seguridad:** La implementación de análisis estático con Snyk permite identificar vulnerabilidades tempranamente en el ciclo de desarrollo.

2. **Calidad del Código:** La cobertura actual del 22.9% indica que se requiere ampliar las pruebas unitarias para cubrir casos edge y flujos alternativos.

3. **Automatización:** La integración con GitHub Actions permite un análisis continuo de seguridad en cada cambio de código.

4. **Contenedorización:** Docker facilita el despliegue consistente de la aplicación en diferentes entornos.

5. **Documentación:** DocFx proporciona una documentación técnica profesional y navegable para el proyecto.

---

## RECOMENDACIONES

1. **Aumentar cobertura de pruebas:** Implementar pruebas adicionales para:
   - Método `Credit()`
   - Casos de error (débitos negativos, montos excesivos)
   - Constructor y validaciones

2. **Reforzar seguridad:** 
   - Revisar y corregir vulnerabilidades identificadas por Snyk
   - Actualizar imagen base de Docker a última versión
   - Implementar análisis de dependencias

3. **Mejorar CI/CD:**
   - Agregar generación automática de reportes HTML como artefactos
   - Implementar publicación automática de documentación
   - Configurar releases automáticas

4. **Optimización:**
   - Reducir tamaño de imagen Docker
   - Implementar caché de compilación
   - Optimizar tiempo de ejecución de tests

---

## ACTIVIDADES COMPLETADAS

- ✅ Configuración de Snyk Cloud
- ✅ Creación de solución .NET con WebAPI
- ✅ Implementación de modelo `BankAccount`
- ✅ Desarrollo de pruebas unitarias con MSTest
- ✅ Generación de reportes de cobertura
- ✅ Creación de Dockerfile funcional
- ✅ Análisis de seguridad con Snyk (código y contenedor)
- ✅ Generación de documentación con DocFx
- ✅ Configuración de GitHub Actions para análisis continuo
- ✅ Integración con GitHub Security (SARIF)

---

## REFERENCIAS

- [Documentación oficial de Snyk](https://docs.snyk.io/)
- [.NET 8 Documentation](https://docs.microsoft.com/dotnet/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [DocFx Documentation](https://dotnet.github.io/docfx/)
- [GitHub Actions](https://docs.github.com/actions)

---

## ANEXOS

### Anexo A: Comandos Principales Ejecutados

```bash
# Instalación de herramientas
scoop install snyk
npm install snyk-to-html -g
dotnet tool install -g docfx
dotnet tool install -g dotnet-reportgenerator-globaltool

# Creación del proyecto
dotnet new sln -o Bank
dotnet new webapi -o Bank.WebApi
dotnet new mstest -o Bank.WebApi.Tests

# Pruebas y cobertura
dotnet test --collect:"XPlat Code Coverage"
ReportGenerator "-reports:./*/*/*/coverage.cobertura.xml" "-targetdir:Cobertura" -reporttypes:MarkdownSummaryGithub

# Docker
docker build -t api-banco .
docker images

# Snyk
snyk auth <TOKEN>
snyk code test --json | snyk-to-html -o code-test-results.html
snyk container test api-banco --json | snyk-to-html -o container-test-result.html

# Documentación
docfx metadata docfx.json
docfx build
```

### Anexo B: Estructura de Archivos de Configuración

- `docfx.json`: Configuración de generación de documentación
- `Dockerfile`: Definición de contenedor multi-stage
- `.github/workflows/snyk.yml`: Pipeline de análisis de seguridad

---

**Fecha de elaboración:** Diciembre 5, 2025  
**Versión del documento:** 1.0



