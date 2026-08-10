# Setup — Herramientas de Diagnóstico DNS

> **Tópico**: 1 — Fundamentos Web
> **Objetivo**: Dominar herramientas de línea de comandos para diagnosticar resolución DNS, verificar registros y troubleshootear problemas de dominios
> **Prerequisito**: Terminal y conexión a internet (WSL o Linux nativo recomendado)

---

## ¿Por qué herramientas DNS?

Cada vez que escribís `google.com` en el navegador, tu máquina necesita traducir ese nombre a una dirección IP. Ese proceso es DNS (Domain Name System) — la "agenda telefónica" de internet. Cuando un dominio no resuelve, una API no responde, o un deploy nuevo no se propaga, **DNS suele ser el culpable**.

Conocer `dig`, `nslookup`, `host` y `whois` te permite diagnosticar problemas que de otra forma serían misteriosos: "¿por qué mi dominio apunta a la IP vieja?", "¿el registro MX está bien configurado?", "¿la propagación DNS ya terminó?". Estas herramientas son tu primer paso de debugging cuando algo no conecta.

---

## Checklist

### 1. Instalar las herramientas

**En Ubuntu/WSL:**
```bash
sudo apt install -y dnsutils whois
```

`dnsutils` incluye `dig`, `nslookup` y `host`. `whois` es un paquete separado.

**En macOS:**
```bash
brew install bind whois
```

- [ ] `dig -v` responde con la versión de BIND
- [ ] `whois --version` responde con una versión

### 2. Usar `dig` para consultar registros DNS

`dig` es la herramienta más completa. Consultá el registro `A` (IPv4) de un dominio:

```bash
dig google.com
```

La respuesta tiene secciones clave:
- **ANSWER SECTION**: las IPs que resuelve el dominio
- **AUTHORITY SECTION**: los nameservers autoritativos
- **ADDITIONAL SECTION**: IPs de los nameservers

Consultá tipos específicos de registros:

```bash
# Registro AAAA (IPv6)
dig google.com AAAA

# Registro MX (servidores de email)
dig google.com MX

# Registro TXT (SPF, DKIM, verificaciones)
dig google.com TXT

# Todos los registros comunes
dig google.com ANY
```

- [ ] Podés ver la ANSWER SECTION con al menos una IP para `google.com`
- [ ] Podés consultar registros MX y ver los servidores de email

### 3. Tracear la delegación DNS con `dig +trace`

Este comando muestra **cada paso** de la resolución, desde los root servers hasta el nameserver autoritativo:

```bash
dig +trace google.com
```

Vas a ver la cadena completa: root servers (.) → TLD servers (.com) → nameservers autoritativos de google.com.

- [ ] El trace muestra al menos 3 niveles de delegación (root, TLD, autoritativo)

### 4. Usar `nslookup` para resolución inversa

`nslookup` es más simple que `dig` pero útil para consultas rápidas:

```bash
# Consulta básica
nslookup google.com

# Usar un DNS server específico (ej: Google DNS 8.8.8.8)
nslookup google.com 8.8.8.8

# Modo interactivo (escribí `server` para cambiar DNS, luego consultás)
nslookup
> server 1.1.1.1
> google.com
> exit
```

Resolución inversa (de IP a nombre):

```bash
nslookup 8.8.8.8
```

- [ ] Podés consultar un dominio usando un DNS server específico (`8.8.8.8` o `1.1.1.1`)
- [ ] Podés hacer resolución inversa de una IP pública

### 5. Usar `host` para consultas rápidas

`host` es la opción más simple — ideal para scripts y verificaciones rápidas:

```bash
# Consulta A (por defecto)
host google.com

# Registro MX
host -t MX google.com

# Resolución inversa
host 8.8.8.8
```

- [ ] `host google.com` devuelve al menos una IP
- [ ] `host -t MX google.com` muestra los servidores de email

### 6. Usar `whois` para información de dominio

`whois` te da información sobre **quién registró** un dominio, cuándo expira, y qué nameservers usa:

```bash
whois google.com
```

Buscá estos campos en la respuesta:
- **Registrar**: quién registró el dominio
- **Creation Date / Expiry Date**: cuándo se creó y cuándo expira
- **Name Server**: los nameservers configurados
- **Status**: estado del dominio (clientTransferProhibited, etc.)

- [ ] Podés ver la fecha de expiración y los nameservers de un dominio público

### 7. Diagnosticar un problema común: propagación DNS

Cuando cambiás los nameservers o un registro A, la propagación puede tardar minutos o horas. Diagnosticá así:

```bash
# Consultá desde diferentes DNS servers
dig @8.8.8.8 midominio.com A     # Google DNS
dig @1.1.1.1 midominio.com A     # Cloudflare DNS
dig @midominio.com A             # Nameserver autoritativo directo
```

Si los resultados difieren, la propagación **no terminó**. El nameserver autoritativo siempre tiene la versión más reciente.

- [ ] Entendés cómo comparar respuestas de diferentes DNS servers para verificar propagación

---

## Verificación

```bash
# 1. dig responde con ANSWER SECTION:
dig google.com A | grep -A 5 "ANSWER SECTION"
# Debería mostrar al menos un registro A

# 2. whois muestra información del dominio:
whois google.com | grep -i "Expiry\|Registrar\|Name Server"
# Debería mostrar al menos uno de estos campos

# 3. host resuelve correctamente:
host google.com | grep "has address"
# Debería mostrar "google.com has address X.X.X.X"
```

**Si podés consultar registros A, MX, TXT de cualquier dominio público Y hacer resolución inversa → herramientas DNS listas. ✅**

---

## Problemas comunes

| Problema | Causa | Solución |
|----------|-------|----------|
| `dig: command not found` | El paquete `dnsutils` no está instalado | `sudo apt install dnsutils` (Ubuntu) o `brew install bind` (macOS) |
| `whois: command not found` | El paquete `whois` no está instalado | `sudo apt install whois` (Ubuntu) o `brew install whois` (macOS) |
| `connection timed out; no servers could be reached` | El DNS server especificado no responde o hay un firewall | Probá con otro DNS (`8.8.8.8` o `1.1.1.1`); verificá tu conexión a internet |
| `SERVFAIL` en la respuesta | El nameserver autoritativo tiene un error de configuración | Verificá los nameservers con `whois`; el problema suele estar en el proveedor de DNS |
| `NXDOMAIN` | El dominio no existe o el registro no existe | Verificá que el dominio esté bien escrito; si existe, puede que el tipo de registro consultado no esté configurado |
| Resultados diferentes entre DNS servers | Propagación DNS en curso | Esperá (puede tardar hasta 48hs); el nameserver autoritativo siempre tiene la versión actual |
| `whois` devuelve "No match" | El dominio no está registrado o el TLD no tiene servidor whois público | Verificá en un registrador como Namecheap o GoDaddy |

---

## Recursos

- [dig — manual](https://linux.die.net/man/1/dig)
- [MDN — DNS](https://developer.mozilla.org/en-US/docs/Glossary/DNS)
- [Cloudflare — What is DNS](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [RFC 1035 — Domain Names Implementation](https://www.rfc-editor.org/rfc/rfc1035)

---

## Preguntas de repaso

- **P:** ¿Cuál es la diferencia principal entre `dig` y `nslookup`?
  **R:** `dig` es más completo y detallado, con secciones separadas (ANSWER, AUTHORITY, ADDITIONAL) y opciones avanzadas como `+trace`. `nslookup` es más simple y tiene un modo interactivo útil para consultas rápidas.

- **P:** ¿Qué muestra `dig +trace` que una consulta normal no muestra?
  **R:** Muestra cada paso de la resolución DNS: desde los root servers (.) hasta los TLD servers (.com, .org, etc.) y finalmente los nameservers autoritativos del dominio. Es ideal para entender la cadena de delegación.

- **P:** Si `dig @8.8.8.8 midominio.com` devuelve una IP vieja pero `dig @ns1.midominio.com midominio.com` devuelve la nueva, ¿qué está pasando?
  **R:** La propagación DNS aún no terminó. El nameserver autoritativo (`ns1.midominio.com`) ya tiene el registro actualizado, pero los DNS caches de Google (`8.8.8.8`) aún tienen la versión anterior. Hay que esperar a que expire el TTL.

- **P:** ¿Para qué sirve el registro TXT y cuándo lo vas a necesitar?
  **R:** Los registros TXT almacenan texto arbitrario y se usan para SPF (anti-spam), DKIM (firma de emails), verificación de dominios (Google Search Console, etc.), y políticas de seguridad como DMARC.

- **P:** ¿Qué significa `NXDOMAIN` en una respuesta DNS?
  **R:** Significa "Non-Existent Domain" — el dominio o el tipo de registro consultado no existe. Puede ser un error de tipeo, un dominio que no fue registrado, o un registro que aún no se configuró.

- **P:** ¿Por qué es útil consultar el mismo dominio desde diferentes DNS servers (`8.8.8.8`, `1.1.1.1`, el autoritativo)?
  **R:** Porque te permite verificar si la propagación DNS terminó. Si todos devuelven el mismo resultado, la propagación es completa. Si difieren, algunos caches aún tienen datos viejos.
