
# Regras Clássicas adicionadas ao NADIA

## Regras implementadas

Até o momento foram adicionadas as seguintes regras clássicas:

### 1. Modus Tollens — `mt`

O Modus Tollens trabalha com implicações.

A ideia da regra é simples: se `P → Q` é verdadeiro, mas `Q` é falso, então `P` também precisa ser falso.

---

### 2. Regra da Disjunção — `ds`

A Regra da Disjunção trabalha com disjunções.

A lógica é: se `P ∨ Q` é verdadeiro, mas um dos lados é falso, então o outro lado obrigatoriamente é verdadeiro.

---

### 3. Introdução da Dupla Negação — `dni`

Essa regra afirma que, se uma proposição é verdadeira, então sua dupla negação também é verdadeira.

A ideia é que, se `P` é verdadeiro, então `¬P` é falso. Logo, `¬¬P` é verdadeiro.

---

### 4. Eliminação da Dupla Negação — `dne`

Essa regra afirma que, se não é verdade que `P` é falso, então `P` é verdadeiro.

Essa é uma regra característica da lógica clássica, pois depende da ideia de que toda proposição é verdadeira ou falsa.

---

Essas regras fazem parte das chamadas **Intelim Rules** (*Introduction/Elimination Rules*) apresentadas no artigo.

---

# Arquitetura do NADIA

O NADIA funciona em várias etapas.

---

## 1. Lexer

O Lexer é responsável por transformar caracteres em tokens reconhecidos pelo sistema.

Exemplo:

```python
self.lexer.add('MODUS_TOLLENS', r'mt')
```

Isso faz com que a sequência de caracteres:

```txt
mt
```

seja reconhecida como o token:

```txt
MODUS_TOLLENS
```

Os tokens representam elementos reconhecíveis da linguagem, como:

- regras (`mt`, `ds`, `dni`, `dne`);
- conectivos (`~`, `&`, `|`);
- números;
- variáveis;
- símbolos auxiliares.

Sem o Lexer, o sistema não conseguiria interpretar o texto da prova.

---

## 2. ParserGenerator

O `ParserGenerator` define quais tokens existem oficialmente na linguagem.

Exemplo:

```python
[
    'MODUS_TOLLENS',
    'DISJUNCTIVE_SYLLOGISM',
    'DOUBLE_NEG_INTROD',
    'DOUBLE_NEG_ELIM'
]
```

Mesmo que o Lexer reconheça um token, o parser ainda precisa saber que ele é válido. Caso contrário, a entrada será rejeitada.

---

## 3. Parser

O Parser define o formato válido das linhas da prova.

Exemplo:

```python
@self.pg.production(
    'step : NUM DOT formula MODUS_TOLLENS NUM COMMA NUM'
)
```

Isso define o seguinte formato:

```txt
linha . fórmula mt linha , linha
```

Exemplo válido:

```txt
3. ~A mt 1,2
```

Outro exemplo:

```python
@self.pg.production(
    'step : NUM DOT formula DOUBLE_NEG_INTROD NUM'
)
```

Formato:

```txt
linha . fórmula dni linha
```

Exemplo:

```txt
2. ~~A dni 1
```

Quando uma linha segue um desses formatos, o parser cria um objeto da regra correspondente.

---

## 4. Classe da Regra

Cada regra possui sua própria classe.

Exemplo:

```python
class ModusTollensDef():
```

As classes possuem:

- armazenamento dos dados da linha;
- lógica da regra;
- validação semântica.

### Estrutura básica

```python
self.line = line
self.formula = formula
self.reference1 = reference1
self.reference2 = reference2
```

Esses atributos armazenam:

- linha atual;
- fórmula da conclusão;
- referências utilizadas.

---

## 5. Método `evaluation()`

O método `evaluation()` é responsável por validar a inferência lógica.

No caso do Modus Tollens:

```txt
P -> Q
~Q
-----
~P
```

A validação ocorre em dois casos:

### Caso 1

```txt
formula1 = P->Q
formula2 = ~Q
conclusão = ~P
```

### Caso 2

```txt
formula2 = P->Q
formula1 = ~Q
conclusão = ~P
```

Isso permite que a ordem das referências seja livre.

Trecho da implementação:

```python
if isinstance(formula1, BinaryFormula) and formula1.is_implication():
    p = formula1.left
    q = formula1.right

    if formula2 == NegationFormula(q) and self.formula == NegationFormula(p):
        valid = True
```

Caso a inferência seja inválida:

```python
parser.has_error = True
deduction_result.add_error(...)
```

---

## 6. Loop de Avaliação

Após o parser processar todas as linhas, o NADIA percorre todas as regras da prova.

Exemplo:

```python
elif isinstance(rule, ModusTollensDef):
    rule.evaluation(self, deduction_result)
```

Nesse momento, o sistema chama `evaluation()` de cada regra para verificar se a inferência está correta.

