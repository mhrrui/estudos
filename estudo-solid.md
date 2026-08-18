# Estudo dos Princípios SOLID

> PDI: [Aprender] [SOLID] Pesquisar e elaborar um documento
> Base de estudo: vídeo do Augusto Galego sobre SOLID (com opiniões de um Dev Sr. sobre a aplicação prática de cada princípio)

## 1. Introdução

SOLID é um acrônimo para 5 princípios de design orientado a objetos, propostos por **Robert C. Martin (Uncle Bob)**. A ideia central é ajudar a escrever código mais limpo e com mais manutenibilidade — ou seja, código mais fácil de alterar no futuro sem quebrar tudo.

Um ponto importante antes de começar: **esses princípios não são consenso absoluto no mercado**. Muitos times aplicam parcialmente, adaptam ou discordam de alguns deles. Isso não os torna inúteis — pelo contrário, entender a *ideia* por trás de cada um ajuda mesmo em códigos que não seguem POO estrito. O segredo é usar com bom senso, não como regra religiosa.

Cada princípio abaixo vem com:
- Uma explicação simples;
- O que dá errado se você não seguir;
- Um exemplo ruim e um exemplo bom em JS/TS;
- Uma opinião prática (o "lado Dev Sr.": quando vale a pena aplicar de verdade, e quando pode ser exagero).

---

## 2. S — Single Responsibility Principle (Princípio da Responsabilidade Única)

**O que diz:** cada classe (ou módulo) deve ter uma única responsabilidade. Assim, se a regra de negócio mudar, existe só um lugar para alterar.

**O que dá errado sem isso:** uma classe vai acumulando funções diferentes. Quando algo precisa mudar, você não sabe mais o impacto — e corre o risco de quebrar uma parte do código ao tentar arrumar outra.

**Exemplo ruim** — uma classe fazendo duas coisas (registrar usuário *e* enviar e-mail):

```ts
class UserService {
  registerUser(name: string, email: string) {
    // salva o usuário no banco
    console.log(`Usuário ${name} registrado.`);

    // e também manda o e-mail de boas-vindas
    console.log(`Enviando e-mail de boas-vindas para ${email}...`);
  }
}
```

**Exemplo bom** — cada responsabilidade na sua própria classe:

```ts
class UserService {
  registerUser(name: string, email: string) {
    console.log(`Usuário ${name} registrado.`);
  }
}

class EmailService {
  sendWelcomeEmail(email: string) {
    console.log(`Enviando e-mail de boas-vindas para ${email}...`);
  }
}
```

Agora, se a lógica de e-mail mudar (por exemplo, trocar o provedor), você só toca em `EmailService`.

**Na prática (opinião do Dev Sr.):** em sistemas pequenos, seguir isso à risca pode gerar boilerplate excessivo — muita pasta, muita classe, muito arquivo para fazer quase nada. O ponto chave é usar bom senso para definir o que realmente é "uma responsabilidade" dentro do domínio da sua aplicação, sem forçar separação artificial.

**Nota do vídeo:** 7/10

---

## 3. O — Open/Closed Principle (Princípio do Aberto/Fechado)

**O que diz:** uma entidade deve estar aberta para *extensão*, mas fechada para *modificação*. Ou seja: para adicionar um comportamento novo, você não deveria precisar editar o código que já existe e já funciona — só estender ele.

**O que dá errado sem isso:** toda vez que surge uma regra nova, alguém abre a mesma classe e vai empilhando `if/else`. Isso aumenta o risco de quebrar uma regra antiga ao adicionar uma nova.

**Exemplo ruim** — regras de desconto direto dentro do checkout:

```ts
class Checkout {
  calculateDiscount(type: string, total: number): number {
    if (type === "christmas") {
      return total * 0.9;
    } else if (type === "blackfriday") {
      return total * 0.7;
    }
    // cada desconto novo = editar essa função de novo
    return total;
  }
}
```

**Exemplo bom** — usando uma abstração de estratégia de desconto:

```ts
interface DiscountStrategy {
  apply(total: number): number;
}

class ChristmasDiscount implements DiscountStrategy {
  apply(total: number): number {
    return total * 0.9;
  }
}

class BlackFridayDiscount implements DiscountStrategy {
  apply(total: number): number {
    return total * 0.7;
  }
}

class Checkout {
  constructor(private discountStrategy: DiscountStrategy) {}

  calculateDiscount(total: number): number {
    return this.discountStrategy.apply(total);
  }
}
```

Para criar um desconto novo, basta criar uma nova classe que implementa `DiscountStrategy` — o `Checkout` nunca precisa ser alterado.

**Na prática (opinião do Dev Sr.):** é um ótimo alvo a perseguir, mas difícil de acertar de primeira. No começo de um projeto (indo de "0 para 1"), é praticamente impossível prever qual vai ser a abstração ideal antes do software evoluir e mostrar seus padrões reais.

**Nota do vídeo:** 8/10

---

## 4. L — Liskov Substitution Principle (Princípio da Substituição de Liskov)

**O que diz:** se uma classe B herda de uma classe A, você deve poder substituir A por B em qualquer lugar do código sem quebrar nada.

**O que dá errado sem isso:** você cria uma herança que parece fazer sentido "no papel", mas na prática a subclasse não consegue cumprir o mesmo contrato da classe pai — e o código quebra em tempo de execução.

**Exemplo prático (analogia com Python, mas o princípio vale igual em TS):** em Python, `bool` é tecnicamente um subtipo de `int` (`True` vale `1`, `False` vale `0`). Uma função que espera dois números continua funcionando perfeitamente se você passar booleanos no lugar — isso é substituição correta.

**Violação clássica** — a subclasse não pode honrar o comportamento da classe pai:

```ts
class Passaro {
  voar(): string {
    return "Estou voando!";
  }
}

class Pinguim extends Passaro {
  voar(): string {
    throw new Error("Pinguins não voam!"); // quebra o contrato da classe pai
  }
}

function fazerPassaroVoar(p: Passaro) {
  console.log(p.voar());
}

fazerPassaroVoar(new Pinguim()); // 💥 quebra em tempo de execução
```

**Correção** — separar o comportamento em uma hierarquia que reflete a realidade:

```ts
class Ave {
  emitirSom(): string {
    return "Som de ave.";
  }
}

class AveVoadora extends Ave {
  voar(): string {
    return "Estou voando!";
  }
}

class Pinguim extends Ave {
  // simplesmente não tem o método voar() — e está tudo bem
}
```

Agora nenhuma subclasse promete um comportamento que não pode cumprir.

**Na prática (opinião do Dev Sr.):** esse é considerado o princípio mais sólido e menos discutível dos cinco — está baseado na teoria dos conjuntos e é essencial para garantir a corretude do código. Diferente do "I" e do "D", aqui não costuma haver muito debate sobre "vale a pena aplicar ou não".

**Nota do vídeo:** 10/10

---

## 5. I — Interface Segregation Principle (Princípio da Segregação de Interface)

**O que diz:** uma classe não deveria ser forçada a depender de métodos que ela não usa. Prefira interfaces pequenas e específicas em vez de uma interface gigante e genérica.

**O que dá errado sem isso:** classes acabam tendo que implementar métodos que não fazem sentido para elas, só porque a interface exige. Isso gera código morto ou métodos que lançam erro "não implementado".

**Exemplo ruim** — uma interface genérica demais:

```ts
interface MetodoPagamento {
  processarCartao(valor: number): void;
  processarPix(valor: number): void;
}

class ProcessadorCartao implements MetodoPagamento {
  processarCartao(valor: number): void {
    console.log(`Processando cartão: R$${valor}`);
  }
  processarPix(valor: number): void {
    throw new Error("Este processador não lida com Pix"); // método forçado e sem sentido
  }
}
```

**Exemplo bom** — quebrando em interfaces menores:

```ts
interface ProcessadorCartao {
  processarCartao(valor: number): void;
}

interface ProcessadorPix {
  processarPix(valor: number): void;
}

class PagamentoCartao implements ProcessadorCartao {
  processarCartao(valor: number): void {
    console.log(`Processando cartão: R$${valor}`);
  }
}

class PagamentoPix implements ProcessadorPix {
  processarPix(valor: number): void {
    console.log(`Processando Pix: R$${valor}`);
  }
}
```

Cada classe implementa só o que realmente usa (isso também é chamado de "composição em vez de herança").

**Na prática (opinião do Dev Sr.):** faz bastante sentido na teoria de POO, mas em muitas codebases reais, tentar seguir isso à risca pode levar à otimização prematura — quebrando interfaces em pedaços pequenos demais antes de existir uma necessidade real. No dia a dia, raramente é a causa de dores de cabeça profundas.

**Nota do vídeo:** 5/10

---

## 6. D — Dependency Inversion Principle (Princípio da Inversão de Dependência)

**O que diz:** módulos de alto nível (regras de negócio) não deveriam depender diretamente de módulos de baixo nível (detalhes técnicos, como banco de dados). Os dois deveriam depender de uma abstração em comum.

**O que dá errado sem isso:** sua regra de negócio fica "amarrada" a uma implementação específica. Trocar essa implementação (ou criar um teste com um mock) fica muito mais difícil.

**Exemplo ruim** — dependendo diretamente de uma implementação concreta:

```ts
class MySQLDatabase {
  save(data: string) {
    console.log(`Salvando no MySQL: ${data}`);
  }
}

class UserService {
  private db = new MySQLDatabase(); // dependência direta e travada

  registerUser(name: string) {
    this.db.save(name);
  }
}
```

**Exemplo bom** — dependendo de uma abstração, injetada por fora:

```ts
interface Database {
  save(data: string): void;
}

class MySQLDatabase implements Database {
  save(data: string) {
    console.log(`Salvando no MySQL: ${data}`);
  }
}

class UserService {
  constructor(private db: Database) {} // injeção de dependência

  registerUser(name: string) {
    this.db.save(name);
  }
}

// uso:
const service = new UserService(new MySQLDatabase());
```

Agora, para testar `UserService`, basta passar um `Database` "falso" (mock) no lugar do MySQL real — sem precisar de um banco de dados de verdade rodando.

**Na prática (opinião do Dev Sr.):** traz ótimo desacoplamento e facilita muito a escrita de testes unitários. Por outro lado, adiciona boilerplate e complexidade na hora de navegar pelo código (você precisa "seguir" a abstração para entender qual implementação está sendo usada de fato). Projetos pequenos raramente trocam a implementação de banco de dados ao longo da vida, então nem sempre o custo compensa.

**Nota do vídeo:** 6.5/10

---

## 7. Como os princípios se conectam

Por trás dos 5 princípios existe uma ideia comum: **baixo acoplamento e alta coesão**.

- **Baixo acoplamento** = as partes do código dependem o mínimo possível umas das outras (isso é o coração do D e do I).
- **Alta coesão** = cada parte do código faz bem uma coisa só, e essa coisa faz sentido junto (isso é o coração do S).
- O **O** e o **L** garantem que você pode estender ou substituir peças do sistema sem quebrar o resto — o que só é possível se as peças já estiverem desacopladas e coesas.

Ou seja: SOLID não são 5 regras independentes, são 5 ângulos diferentes do mesmo objetivo.

---

## 8. Checklist de bolso

Ao revisar um código seu (ou de outra pessoa), pergunte:

- [ ] **S** — Essa classe/função faz mais de uma coisa? Ela teria mais de um motivo para mudar no futuro?
- [ ] **O** — Se eu precisar adicionar um comportamento novo, vou precisar editar esse código ou consigo só estender ele?
- [ ] **L** — Se eu trocar essa classe pai por uma subclasse dela, algo quebra ou se comporta de forma inesperada?
- [ ] **I** — Essa classe é obrigada a implementar métodos que ela não usa de verdade?
- [ ] **D** — Essa classe está "presa" a uma implementação concreta (ex: um banco específico), ou depende de uma abstração?

**Importante:** nenhum desses princípios é uma regra absoluta. Em código pequeno ou pouco propenso a mudar, forçar SOLID em todo lugar pode gerar mais complexidade do que resolve. O objetivo é usar como lente de análise, não como checklist obrigatório item a item.

---

## 9. Referências

- Augusto Galego — vídeo sobre Princípios SOLID (YouTube): https://www.youtube.com/watch?v=2yqHlJ2HbTo
- Robert C. Martin (Uncle Bob) — criador original dos princípios SOLID
