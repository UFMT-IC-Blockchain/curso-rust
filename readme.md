# Curso de rust
Este repositório contém exemplos práticos e exercícios utilizados durante o curso de Rust. O objetivo é fornecer uma base sólida desde cálculos simples até o desenvolvimento de Smart Contracts utilizando Soroban SDK.

---
## Codigos a serem executados no Rust Playground

### Código 1 - Calculos basicos

Rodar em Rust Playground: 
https://play.rust-lang.org/?version=stable&mode=debug&edition=2024

```rust
// Exibe informações básicas de perfil do usuário
fn exibir_dados_usuario() {
    let idade: i32 = 42;
    let pontos = 100;
    println!("Idade: {}, Pontos acumulados: {}", idade, pontos);
}

// Demonstra operações aritméticas básicas (subtração, multiplicação, divisão e resto)
fn calcular_financas() {
    let receita = 10_000; // O '_' facilita a leitura de números grandes
    let despesas = 3_000;

    println!("Lucro: {}", receita - despesas);
    println!("Dobro da receita: {}", receita * 2);
    println!("Quociente: {}", receita / despesas);
    println!("Resto da divisão: {}", receita % despesas);
}

// Realiza uma comparação lógica (booleana) entre dois valores
fn comparar_metas() {
    let meta_atingida = 5 + 5;
    let meta_esperada = 10;
    // O operador '==' retorna true ou false
    println!("Meta atingida: {} = Meta esperada: {} ? {}", meta_atingida, meta_esperada, meta_atingida == meta_esperada);
}

// Utilitário visual para separar os blocos no terminal
fn linha() {
    println!("----------------------------")
}

fn main() {
    linha();
    exibir_dados_usuario();
    linha();
    calcular_financas();
    linha();
    comparar_metas();
    linha();
}
`````	
---

### Código 2 - Calculo de media

Rodar em Rust Playground: 
https://play.rust-lang.org/?version=stable&mode=debug&edition=2024

```rust
// Calcula a média das notas fornecidas
fn calcular_media(n1: f64, n2: f64, n3: f64) -> f64 {
    let soma :f64 = n1 + n2 + n3;
    let qnt_notas :i32 = 3;
    let media = soma / qnt_notas;
    media
}

// Define o critério de aprovação baseado na média
fn foi_aprovado(media: f64) -> bool {
    let foi_aprovado = media <= 7.0 // Lógica de aprovação para notas menores ou iguais a 7
    
    foi_aprovado
}

// Exibe o status final do aluno
fn mostrar_resultado(media: f64, aprovado: bool) {
    println!("Média: {:.2}", media);

    // Tenta comparar um booleano com uma string (Erro de tipo)
    if aprovado == "sim" {
        println!("Aprovado");
    } else {
        println!("Reprovado");
    }
}

fn main() {
    let nota1 = 3.5;
    let nota2 = 7.88;
    let nota3 = 9.75;

    // Processamento dos dados através das funções auxiliares
    let media = calcular_media(nota1, nota2, nota3);
    let aprovado = foi_aprovado(media);

    mostrar_resultado(media, aprovado);
}
`````	
	
---

### Código 3 - Verificar idade e imprimir status

Rodar em Rust Playground: 
https://play.rust-lang.org/?version=stable&mode=debug&edition=2024

#### Objetivo
O objetivo do código é: se a idade for 18 ou mais, imprime "Maior", se não, imprime "Menor".

```rust
fn main() {
    // Define o valor da idade para a verificação
    let idade = 16;

    // Inicia a estrutura condicional para determinar a categoria
    if idade > 18 {
        // Atribui o rótulo de maioridade à variável status
        let status = "Maior"
    } else {
        // Altera o rótulo para indicar que é menor de idade
        status = "Menor";
    }

    // Exibe a mensagem final com o status processado no bloco anterior
    println!("Você é: {}", status);
}
`````
	
---


### Código 4 - SmartContract

Rodar em https://soropg.com/

#### Arquivo lib.rs

```rust
// Indica que o contrato não usará a biblioteca padrão (necessário para o ambiente WASM)
#![no_std]
use soroban_sdk::{contract, contractimpl, log, symbol_short, Env, Symbol};

// Define uma constante de símbolo curto para identificar a chave no armazenamento
const COUNTER: Symbol = symbol_short!("COUNTER");

#[contract]
pub struct IncrementContract;

#[contractimpl]
impl IncrementContract {
    // Função para aumentar o valor do contador e estender o tempo de vida do dado
    pub fn increment(env: Env) -> u32 {
        // Recupera o valor atual ou define como 0 caso não exista
        let mut count: u32 = env.storage().instance().get(&COUNTER).unwrap_or(0);
        log!(&env, "count: {}", count);

        count += 1;
        // Salva o novo valor no armazenamento de instância do contrato
        env.storage().instance().set(&COUNTER, &count);
        // Estende o Time-to-Live (TTL) para evitar que o dado seja arquivado
        env.storage().instance().extend_ttl(50, 100);

        count
    }

    // Função para diminuir o valor do contador
    pub fn decrement(env: Env) -> u32 {
        // Recupera o valor atual da chave COUNTER
        let mut count: u32 = env.storage().instance().get(&COUNTER).unwrap_or(0);
        log!(&env, "count: {}", count);

        count -= 1;
        // Atualiza o valor decrementado no armazenamento
        env.storage().instance().set(&COUNTER, &count);
        // Garante a persistência dos dados por mais tempo
        env.storage().instance().extend_ttl(50, 100);

        count
    }
}

// Declaração do módulo de testes
mod test;
`````

#### Arquivo test.rs

```rust
// Indica que este módulo só deve ser compilado quando executarmos os testes
#![cfg(test)]

// Importa o contrato e o cliente gerado automaticamente para interação
use crate::{IncrementContract, IncrementContractClient};
use soroban_sdk::Env;

#[test]
fn test() {
    // Inicializa o ambiente de simulação do Soroban
    let env = Env::default();
    
    // Registra o contrato no ambiente de teste e obtém um ID único
    let contract_id = env.register(IncrementContract, ());
    
    // Cria um cliente para chamar as funções do contrato de forma simplificada
    let client = IncrementContractClient::new(&env, &contract_id);

    // Valida se a função de incremento está somando corretamente (1, 2, 3)
    assert_eq!(client.increment(), 1);
    assert_eq!(client.increment(), 2);
    assert_eq!(client.increment(), 3);
    
    // Valida se a função de decremento está subtraindo os valores (de volta para 2 e 1)
    assert_eq!(client.decrement(), 2);
    assert_eq!(client.decrement(), 1);
}
`````
	
---
