# Calculadora ANTLR4 - Análisis de Precedencia y Asociatividad

## Estructura del Proyecto

```
proyecto/
├── calculadora.py          # Aplicación principal
├── EvalVisitor.py         # Evaluador de expresiones (patrón Visitor)
├── labeldExpr.g4         # Gramática estándar
├── labeldExpr_alt.g4     # Gramática con reglas modificadas
└── tests.txt      # Expresiones de ejemplo
```

## Componentes Principales

### calculadora.py - Aplicación Central
El núcleo del programa que gestiona la carga dinámica de gramáticas:

```python
def load_parser_and_lexer(use_alt):
    if use_alt:
        lexer_mod = 'labeldExpr_altLexer'
        parser_mod = 'labeldExpr_altParser'
    else:
        lexer_mod = 'labeldExprLexer'
        parser_mod = 'labeldExprParser'
```

**Características destacadas:**
- Intercambio dinámico entre gramáticas usando el flag `--alt`
- Procesamiento de archivos o entrada directa desde consola
- Validación automática de módulos ANTLR generados

### EvalVisitor.py - Motor de Evaluación
Implementa el patrón Visitor para recorrer y evaluar el árbol de sintaxis abstracta:

```python
class EvalVisitor(labeldExprVisitor):
    def __init__(self):
        self.memory = {}          # Almacén de variables
        self.use_degrees = True   # Ángulos en grados por defecto
```

**Operaciones soportadas:**
- **Aritméticas básicas:** Suma, resta, multiplicación, división con protección contra división por cero
- **Potenciación:** Usando `math.pow()` para mayor precisión
- **Factorial:** Implementado con `math.factorial()`
- **Funciones trigonométricas:** sin(), cos(), tan() con conversión automática grados/radianes
- **Logarítmicas:** ln() (natural) y log() (base 10)
- **Raíz cuadrada:** sqrt()
- **Variables:** Asignación y recuperación de valores

##  Gramáticas Implementadas

### Gramática Estándar (labeldExpr.g4)
Sigue las reglas matemáticas convencionales:

```antlr
expr
  : expr POW expr            #Pow
  | expr op=(MUL | DIV) expr #MulDiv
  | expr op=(ADD | SUB) expr #AddSub
  | expr FACT                #Fact
  | SUB expr                 #UnaryMinus
  | ID '(' expr ')'          #Function
  | INT | DOUBLE | ID        #Literales
  | '(' expr ')'             #Parens
  ;
```

**Precedencia (mayor → menor):**
1. Paréntesis y llamadas a función
2. Operador unario menos (-)
3. Factorial (!)
4. Exponenciación (^)
5. Multiplicación y división (*, /)
6. Suma y resta (+, -)

**Asociatividad:** Izquierda para todos los operadores binarios

### Gramática Alternativa (labeldExpr_alt.g4)
Versión experimental con reglas modificadas:

```antlr
expr
  : expr POW expr                            #Pow
  | <assoc=right> expr op=(ADD | SUB) expr   #AddSub
  | expr op=(MUL | DIV) expr                 #MulDiv
  | expr FACT                                #Fact
  | SUB expr                                 #UnaryMinus
  | ID '(' expr ')'                          #Function
  | INT | DOUBLE | ID                        #Literales
  | '(' expr ')'                             #Parens
  ;
```

**Cambios principales:**
- Suma y resta con **asociatividad derecha**
- **Precedencia alterada:** Suma/resta tienen mayor prioridad que multiplicación/división

## Instalación y Configuración

### Dependencias
```bash
pip install antlr4-python3-runtime
```

### Generación de Analizadores
```bash
# Gramática estándar
antlr4 -Dlanguage=Python3 -visitor labeldExpr.g4

# Gramática alternativa
antlr4 -Dlanguage=Python3 -visitor labeldExpr_alt.g4
```

##  Uso de la Aplicación

### Ejecución con gramática estándar:
```bash
python calculadora.py tests.txt
```

### Ejecución con gramática alternativa:
```bash
python calculadora.py --alt tests.txt
```

##  Casos de Prueba

Contenido del archivo `tests.txt`:
```
2+2*2
5+5+5
10-4-3
4^2^3
3!
sin(90)
```

### Resultados Comparativos

| Expresión | Gramática Estándar | Gramática Alternativa | Explicación |
|-----------|-------------------|----------------------|-------------|
| `2+2*2`   | 6                 | 8                    | Precedencia: `2+(2*2)` vs `(2+2)*2` |
| `5+5+5`   | 15                | 15                   | Mismo resultado independiente de asociatividad |
| `10-4-3`  | 3                 | 9                    | Asociatividad: `(10-4)-3` vs `10-(4-3)` |
| `4^2^3`   | 4096              | 4096                 | Exponenciación mantiene asociatividad derecha |
| `3!`      | 6                 | 6                    | Factorial no se ve afectado |
| `sin(90)` | 1.0               | 1.0                  | Funciones trigonométricas inalteradas |


## Pruebas

<img width="365" height="403" alt="imagen" src="https://github.com/user-attachments/assets/6f44d5de-28ba-4984-a494-1e23e3bdae66" />
