# KMS — Envelope Encryption APIs

## El proceso (concepto)

1. Clave **raíz/maestra (CMK)** vive en KMS, nunca sale.
2. `GenerateDataKey` → data key en **texto plano** + data key **cifrada**.
3. Cifrás los **datos** con la **data key en texto plano** (local, cualquier tamaño).
4. Borrás la data key en texto plano de memoria.
5. Guardás la data key **cifrada** al lado de los datos.
6. Descifrar: mandás la data key cifrada a KMS → la descifra con la raíz → la usás.

**Frase clave**: datos ← data key (texto plano); data key ← clave RAÍZ (texto plano).

### Concepto que confunde

- Una clave DEBE estar en **texto plano** para poder cifrar/descifrar algo.
  Una "clave cifrada" no cifra nada (primero hay que descifrarla).
- La **raíz** protege a la data key. La data key NUNCA cifra a la raíz (eso está invertido).
- Correcto: "data key cifra datos, **clave raíz en texto plano** cifra la data key".
- Trampa: "cifrar la data key con una clave CIFRADA de nivel superior" → MAL.

| API                               | Devuelve                                 | Cuándo                                                 |
| --------------------------------- | ---------------------------------------- | ------------------------------------------------------ |
| `GenerateDataKey`                 | Plaintext + Encrypted                    | Cifrar **AHORA**                                       |
| `GenerateDataKeyWithoutPlaintext` | Solo Encrypted                           | Cifrar **DESPUÉS** (pedís Decrypt cuando lo necesites) |
| `Encrypt`                         | Cifra datos pequeños (**<4 KB**) directo | Secreto chico, NO archivos                             |
| `Decrypt`                         | Plaintext de una key cifrada             | Recuperar plaintext para usarlo                        |
| `GenerateRandom`                  | Bytes aleatorios                         | Randomness, NO cifrado                                 |

## Disparadores

- "**ahora**" / "cifrar inmediatamente" → `GenerateDataKey`
- "**en un momento posterior**" / "después" / "guardar solo cifrada" → `GenerateDataKeyWithoutPlaintext`
- "datos pequeños" / "<4 KB" → `Encrypt`

## Trampa

`WithoutPlaintext` parece "no me sirve" pero ES la opción **más segura cuando no cifrás YA**. Plaintext vive en memoria solo cuando lo necesitás (vía Decrypt).

## Anti-cruce de números (error propio)

- **KMS `Encrypt` directo = 4 KB** máx.
- **400 KB = tamaño máx de un ÍTEM en DynamoDB** (NO es KMS).
- No los cruces: dato > 4 KB → envelope encryption (`GenerateDataKey`).

## Pregunta de prueba

Necesitás cifrar un archivo de 2 GB del lado de tu aplicación usando KMS. ¿Qué API?

A) `Encrypt` directo
B) `GenerateDataKey`
C) `GenerateRandom`
D) `Decrypt`

<details><summary>Respuesta</summary>

**B** (`GenerateDataKey`): te da la data key en texto plano (cifrás local) + cifrada
(la guardás). Envelope encryption, sin límite de tamaño.
Cuándo sería cada una:
- **Encrypt directo** → solo datos < 4 KB.
- **GenerateRandom** → bytes aleatorios, no cifra.
- **Decrypt** → recuperar el plaintext de una data key cifrada (al descifrar).
</details>
