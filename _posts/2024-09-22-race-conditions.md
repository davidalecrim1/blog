---
layout: post
title:  "Race conditions, ou condições de corrida. Você ouviu falar?"
author: david
categories: [ Backend, Banco de Dados, Engenharia de Software]
image: assets/images/posts/race-condition.webp
comments: true
featured: true
hidden: true

---

Todo desenvolvedor, em algum momento, precisará entender e saber as implicações das race conditions no desenvolvimento de aplicações, especialmente no backend ao interagir com bancos de dados.

Imagine sua conta bancária com saldo de R$ 100. Agora, você recebe um PIX no valor de R$ 200 e, no mesmo instante, realiza uma compra no débito de R$ 50. Se ambas as operações acessarem o saldo ao mesmo tempo, e a race condition não for tratada na aplicação e no banco de dados, o saldo poderia resultar em um valor incorreto. A primeira operação consulta o saldo de R$ 100 e, após o crédito, resulta em R$ 300, finalizando a operação. Já a segunda operação, ocorrendo simultaneamente, também lê o saldo de R$ 100 e, após o débito, resulta em R$ 50 de saldo definitivo na conta, desconsiderando o crédito anterior e causando uma inconsistência no saldo persistido no banco de dados.

As race conditions ocorrem quando múltiplas operações acessam e modificam dados simultaneamente, sem a devida sincronização. O resultado pode ser imprevisível, causando erros como dados inconsistentes no banco ou falhas no sistema.

Para resolver esse problema, existem duas abordagens comuns: a **gestão otimista** e a **gestão pessimista** de race conditions.


### 📊 Gestão Pessimista

Na gestão pessimista, a aplicação assume que as operações concorrentes causarão conflitos, então ele bloqueia o recurso compartilhado até que uma operação finalize seu uso. Esse tipo de controle garante que, enquanto uma transação está sendo processada, nenhuma outra pode acessar o recurso, eliminando a possibilidade de race conditions.

Exemplo em SQL:

```sql
BEGIN;
SELECT saldo FROM contas WHERE id = 1 FOR UPDATE;
-- Atualiza o saldo com base na lógica de crédito ou débito
UPDATE contas SET saldo = saldo + 200 WHERE id = 1;
COMMIT;
```

No exemplo acima, o comando `FOR UPDATE` garante que o saldo da conta está bloqueado para outras transações enquanto a operação atual está sendo executada, evitando que outra transação leia ou escreva no mesmo registro até que o `COMMIT` seja realizado.

### 🚀 Gestão Otimista

Na gestão otimista, a aplicação assume que conflitos são raros e permite que as operações concorram pelo recurso sem bloqueios explícitos. Quando uma operação é finalizada, a aplicação verifica se o dado foi modificado durante a transação. Caso tenha sido, a operação falha e a aplicação deve tentar novamente ou abortar.

A gestão otimista é mais eficiente em cenários onde o conflito de concorrência é raro, já que as transações podem ser executadas simultaneamente sem bloqueios. Contudo, ela exige um mecanismo de controle de versões ou verificação de mudanças para detectar conflitos.

Exemplo com controle de versão (usando uma coluna `versao` no banco):

```sql
BEGIN;
SELECT saldo, versao FROM contas WHERE id = 1;
-- Verifica se a versão é a mesma antes de atualizar
UPDATE contas SET saldo = saldo + 200, versao = versao + 1 WHERE id = 1 AND versao = 1;
COMMIT;
```

Aqui, a coluna `versao` é usada para garantir que a transação só será bem-sucedida se o registro não foi alterado por outra operação concorrente. Caso a `versao` tenha mudado, a transação falha e pode ser repetida ou abortada.

### Exemplo em Go com SQL otimista

No Go, podemos implementar a estratégia de controle otimista ao usar uma função que tenta realizar a operação e lida com o erro caso haja um conflito:

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    _ "github.com/lib/pq"
)

func updateSaldo(db *sql.DB, id int, amount int) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()

    var saldo int
    var versao int

    err = tx.QueryRow("SELECT saldo, versao FROM contas WHERE id = $1", id).Scan(&saldo, &versao)
    if err != nil {
        return err
    }

    // Tenta atualizar o saldo e a versão
    res, err := tx.Exec("UPDATE contas SET saldo = $1, versao = versao + 1 WHERE id = $2 AND versao = $3", saldo+amount, id, versao)
    if err != nil {
        return err
    }

    rowsAffected, err := res.RowsAffected()
    if err != nil {
        return err
    }

    if rowsAffected == 0 {
        return fmt.Errorf("Conflito de versão, tente novamente")
    }

    return tx.Commit()
}

func main() {
    connStr := "user=postgres dbname=teste sslmode=disable"
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        log.Fatal(err)
    }

    err = updateSaldo(db, 1, 200)
    if err != nil {
        fmt.Println("Erro ao atualizar saldo:", err)
    } else {
        fmt.Println("Saldo atualizado com sucesso")
    }
}
```

No exemplo acima, utilizamos a abordagem otimista. Caso a versão do registro tenha sido modificada por outra transação, a aplicação retorna um erro, indicando que a operação precisa ser repetida.

### Conclusão

Race conditions podem ser críticas ao desenvolver sistemas concorrentes e, para tratá-las, é essencial escolher a estratégia correta. A gestão pessimista funciona bem quando há uma alta probabilidade de conflitos, já que bloqueia o acesso ao recurso. Já a gestão otimista é mais eficiente quando os conflitos são raros, permitindo maior paralelismo nas operações.

Ao lidar com banco de dados e sistemas de alta concorrência, sempre considere a melhor abordagem para garantir a integridade e consistência dos dados.