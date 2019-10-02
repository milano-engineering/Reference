# Implementar Log4net en un projecto CS
## 1. REFERENCIAR la DLL log4net
  *... Más detalles acá de versionado y NuGet ???*

## 2. En Global.asax

````csharp
protected void Application_Start(object sender, EventArgs e)
{
	log4net.Config.XmlConfigurator.Configure();
}
````
  *... Cómo hacerlo efectivamente con el atributo propiedades ???*

## 3. En el web.config
```xml  
<configSections>
	<section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net" />
</configSections>
	
<log4net>
    <appender name="FileAppender" type="log4net.Appender.RollingFileAppender">
      <file value="D:\LOGS\FASCServicesXYZ-1.log" />
      <appendToFile value="true" />
      <rollingStyle value="Size" />
      <maxSizeRollBackups value="5" />
      <maximumFileSize value="10MB" />
      <staticLogFileName value="true" />
      <layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%date [%thread] %level %logger - %message%newline" />
      </layout>
    </appender>

	<!-- DATABASE APPENDER -->
    <appender name="AdoNetAppender" type="log4net.Appender.AdoNetAppender">
      <bufferSize value="1" />
      <connectionType value="System.Data.SqlClient.SqlConnection, System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
      <connectionStringName value="log4netConnectionString" />
      <commandText value="INSERT INTO dbo.FinandinaTasasService ([Date],[Thread],[Level],[Logger],[Message],[Exception]) VALUES (@log_date, @thread, @log_level, @logger, @message, @exception)" />
      <parameter>
        <parameterName value="@log_date" />
        <dbType value="DateTime" />
        <layout type="log4net.Layout.RawTimeStampLayout" />
      </parameter>
      <parameter>
        <parameterName value="@thread" />
        <dbType value="String" />
        <size value="255" />
        <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="%thread" />
        </layout>
      </parameter>
      <parameter>
        <parameterName value="@log_level" />
        <dbType value="String" />
        <size value="50" />
        <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="%level" />
        </layout>
      </parameter>
      <parameter>
        <parameterName value="@logger" />
        <dbType value="String" />
        <size value="255" />
        <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="%logger" />
        </layout>
      </parameter>
      <parameter>
        <parameterName value="@message" />
        <dbType value="String" />
        <size value="4000" />
        <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="%message" />
        </layout>
      </parameter>
      <parameter>
        <parameterName value="@exception" />
        <dbType value="String" />
        <size value="2000" />
        <layout type="log4net.Layout.ExceptionLayout" />
      </parameter>
    </appender>
	
    <root>
      <level value="All" />
      <appender-ref ref="FileAppender" />
	  <appender-ref ref="AdoNetAppender" />
    </root>
	
</log4net>
```  
## 4. Crear la cadena de conexión para LOGEAR	
```xml
<connectionStrings>
	<add name="log4netConnectionString" connectionString="Data Source=FABOGDESQABD\DESARROLLO;Initial Catalog=LOG4NET;User ID=sa; Password=Finandina2014" providerName="System.Data.SqlClient" />
```
## 5. Crear la tabla 
```sql
CREATE TABLE [dbo].[FinandinaTasasService](  
	[Id] [int] IDENTITY(1,1) NOT NULL,  
	[Date] [datetime] NOT NULL,  
	[Thread] [varchar](255) NOT NULL,  
	[Level] [varchar](50) NOT NULL,  
	[Logger] [varchar](255) NOT NULL,  
	[Message] [varchar](4000) NOT NULL,  
	[Exception] [varchar](2000) NULL,  
 CONSTRAINT [PK_FinandinaTasasService] PRIMARY KEY CLUSTERED   
([Id] ASC) WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]) ON [PRIMARY] 
```

## 6.  LOG4NET LOGIN DEBUG en el web.config
```xml
<appSettings>
    <add key="log4net.Internal.Debug" value="true"/>
</appSettings>

<system.diagnostics>
    <trace autoflush="true">
      <listeners>
        <add
            name="textWriterTraceListener"
            type="System.Diagnostics.TextWriterTraceListener"
            initializeData="D:\logs\log4net.log" />
      </listeners>
    </trace>
</system.diagnostics>
```

## 7. Como se loggea
```csharp
public Class MyClase {
	private static readonly log4net.ILog log = log4net.LogManager.GetLogger(System.Reflection.MethodBase.GetCurrentMethod().DeclaringType);
  
	public List<TasasBO> ObtenerTasasVehiculo(List<TasasBO> infoFiltro)
        {

            var result = new List<TasasBO>();
            var errorMessage = String.Empty;

            try
            {
	      \\ ...
            }
            catch (Exception e)
            {
                errorMessage = String.Format(
                    "{0}, InnerException: {1}",
                    e.Message,
                    e.InnerException == null ? "?" : e.InnerException.Message);
            }

            log.DebugFormat(
                "ObtenerTasasVehiculo Request: {0} Response. {1} Error: {2}",
                JsonConvert.SerializeObject(infoFiltro),
                JsonConvert.SerializeObject(result),
                errorMessage);

            return result;
        }
}
 ``` 
  
