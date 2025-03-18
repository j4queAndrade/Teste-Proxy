# Padrão Proxy

## Intenção
O padrão Proxy fornece um substituto ou representante de um objeto para controlar o acesso a ele. O Proxy atua como um intermediário entre o cliente e o objeto real, permitindo que o cliente interaja com o objeto sem precisar conhecer sua implementação interna

## Também conhecido como
Surrogate

## Motivação

## Aplicabilidade
O padrão Proxy é aplicável em várias situações, incluindo:


## 1. Proxy Remoto
Este tipo de Proxy é utilizado quando um objeto está em um espaço de endereçamento diferente, como um servidor remoto. Ele atua como um intermediário, permitindo que o cliente interaja com o objeto como se estivesse localmente.
### Código
```java
interface ServicoRemoto {
    String obterDados();
}

class ServicoRemotoReal implements ServicoRemoto {
    @Override
    public String obterDados() {
        return "Dados do serviço remoto.";
    }
}

class ProxyRemoto implements ServicoRemoto {
    private ServicoRemotoReal servicoReal;

    @Override
    public String obterDados() {
        if (servicoReal == null) {
            servicoReal = new ServicoRemotoReal();
        }
        return servicoReal.obterDados();
    }
}

public class ExemploProxyRemoto {
    public static void main(String[] args) {
        ServicoRemoto servico = new ProxyRemoto();
        System.out.println(servico.obterDados());
    }
}

```
## 2. Proxy Virtual
Este Proxy é usado para criar objetos pesados sob demanda, economizando recursos ao evitar a criação de instâncias até que sejam realmente necessárias.
### Código
```java
interface Imagem {
    void exibir();
}

class ImagemReal implements Imagem {
    private String nome;

    public ImagemReal(String nome) {
        this.nome = nome;
        carregarImagem();
    }

    private void carregarImagem() {
        System.out.println("Carregando a imagem: " + nome);
    }


    @Override
    public void exibir() {
        System.out.println("Exibindo a imagem: " + nome);
    }
}

class ProxyImagem implements Imagem {
    private ImagemReal imagemReal;
    private String nome;

    public ProxyImagem(String nome) {
        this.nome = nome;
    }


    @Override
    public void exibir() {
        if (imagemReal == null) {
            imagemReal = new ImagemReal(nome);
        }
        imagemReal.exibir();
    }
}

public class ExemploProxyVirtual {
    public static void main(String[] args) {
        Imagem imagem = new ProxyImagem("foto.jpg");
        imagem.exibir(); // Carrega e exibe a imagem
        imagem.exibir(); // Apenas exibe a imagem, sem recarregar
    }
}
```
## 3. Proxy de Proteção
Este Proxy é projetado para controlar o acesso a um objeto, garantindo que apenas usuários autorizados possam interagir com ele. É especialmente útil em sistemas que lidam com informações sensíveis.
### Código
```java
public interface ContaBancaria {
    void verSaldo();
}

public class ContaBancariaReal implements ContaBancaria{
    private String titular;  
    private double saldo;

    public ContaBancariaReal(String titular, double saldo) {
        this.titular = titular;
        this.saldo = saldo;
    }

    @Override
    public void verSaldo() {
        System.out.printf("Exibindo saldo da conta de %s: R$ %.2f%n", titular, saldo);;
    }
}

public class ProxySeguranca implements ContaBancaria{
    private ContaBancaria contaReal;
    private String papelUsuario;

    public ProxySeguranca(String titular, double saldo, String papelUsuario) {
        this.contaReal = new ContaBancariaReal(titular, saldo);
        this.papelUsuario = papelUsuario;
    }

    @Override
    public void verSaldo() {
        if ("ADMIN".equals(papelUsuario)) {
            contaReal.verSaldo();
        }
        else {
            System.out.println("Acesso negado. Você não tem permissão.");
        }
    }
}

public class ExemploProxy {
    public static void main(String[] args) {
        ContaBancaria usuarioComum = new ProxySeguranca("Pedro", 700.00, "USUARIO");
        usuarioComum.verSaldo();

        ContaBancaria admin = new ProxySeguranca("Joana", 2000.00,"ADMIN");
        admin.verSaldo();
    }
}

```

## 4. Smart Reference (Referência Inteligente)
Este Proxy é utilizado para gerenciar referências a objetos, permitindo que ações adicionais sejam executadas ao acessar um objeto, como contar referências ou carregar objetos persistentes.
### Código

## Estrutura
![image](https://github.com/user-attachments/assets/6b155c4b-406a-472c-b763-da018a518681)


## Participantes:
- Subject (Arquivo): Define a interface comum para RealSubject e Proxy.
- RealSubject (ArquivoReal): Implementa o comportamento real.
- Proxy (ProxyArquivo): Controla o acesso a RealSubject, podendo armazenar referências e gerenciar chamadas.

## Colaborações:
- O Proxy gerencia o acesso ao RealSubject, podendo delegar chamadas ou adicionar funcionalidade extra.

## Consequências:
- Controle sobre a criação do objeto: O objeto real só é instanciado quando necessário.
- Melhoria no desempenho: Pode reduzir chamadas a recursos dispendiosos (ex: carregamento de arquivos remotos).
- Segurança e acesso controlado: Pode restringir acesso com autenticação e permissões.

## Implementação:

## Exemplo:

Classe Arquivo - Subject:
```java
package proxy;

public interface Arquivo {
    void carregar();
    void exibir();
}
```

Classe ArquivoReal - RealSubject:
```java
package proxy;

public class ArquivoReal implements Arquivo {
    private String nome;

    public ArquivoReal(String nome) {
        this.nome = nome;
        carregar();
    }

    @Override
    public void carregar() {
        System.out.println("Carregando arquivo: " + nome);
    }

    @Override
    public void exibir() {
        System.out.println("Exibindo arquivo: " + nome);
    }
}
```

Classe ProxyArquivo - Proxy:
```java
package proxy;

public class ProxyArquivo implements Arquivo {
    private ArquivoReal arquivoReal;
    private String nome;

    public ProxyArquivo(String nome) {
        this.nome = nome;
    }

    @Override
    public void carregar() {
        if (arquivoReal == null) {
            arquivoReal = new ArquivoReal(nome);
        }
    }

    @Override
    public void exibir() {
        carregar();
        arquivoReal.exibir();
    }
}
```

Classe Main - Cliente:
```java
package proxy;

public class Main {
    public static void main(String[] args) {
        Arquivo arquivo = new ProxyArquivo("documento.pdf");
        arquivo.exibir(); // Carrega e exibe
        System.out.println("---");
        arquivo.exibir(); // Apenas exibe, sem carregar novamente
    }
}
```

## Usos conhecidos:
**Serviços remotos:** Representa objetos remotos em aplicações distribuídas.

**Autenticação e segurança:** Restringe acesso a objetos sensíveis.
Cache de imagens e arquivos: Evita carregamento repetitivo de recursos.

**Controle de acesso a sistemas complexos:** Garante que apenas usuários autorizados interajam com determinados objetos.

## Padrões relacionados:
**- Decorator:** Embora similar ao Proxy, o Decorator adiciona funcionalidades sem restringir acesso.

**- Adapter:** Converte interfaces sem atuar como intermediário.


## Conclusão
O padrão Proxy é uma ferramenta que oferece flexibilidade e controle em sistemas de software. Ao permitir que um objeto atue como um substituto para outro, ele facilita o gerenciamento de acesso, otimiza o uso de recursos e promove um design modular. Com suas diversas aplicações, desde Proxies Remotos até Proxies de Proteção e Referências Inteligentes. 

## Referências
GAMMA, Erich; HELM, Richard; JOHNSON, Ralph; VLISSIDES, John. Padrões de projeto: soluções reutilizáveis de software orientado a objetos. 1. ed. Porto Alegre: Bookman, 2000.

