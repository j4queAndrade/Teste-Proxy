# Padrão Proxy

## Intenção
O padrão Proxy fornece um substituto ou representante de um objeto para controlar o acesso a ele. O Proxy atua como um intermediário entre o cliente e o objeto real, permitindo que o cliente interaja com o objeto sem precisar conhecer sua implementação interna

## Também conhecido como
Surrogate

## Motivação
Em sistemas que gerenciam dados sensíveis, como informações de saúde, o acesso deve ser controlado para garantir a privacidade. O padrão Proxy atua como intermediário, verificando permissões antes de permitir o acesso aos dados. Além disso, ele pode implementar caching para otimizar o desempenho, garantindo que apenas usuários autorizados acessem informações críticas.

## Exemplo Aplicado
```mermaid
classDiagram
    class SistemaSaude {
        +visualizarDados(paciente: String)
        +editarDados(paciente: String, novosDados: String)
    }

    class SistemaSaudeReal {
        +visualizarDados(paciente: String)
        +editarDados(paciente: String, novosDados: String)
    }

    class ProxySistemaSaude {
        -sistemaReal: SistemaSaudeReal
        -usuario: String
        +visualizarDados(paciente: String)
        +editarDados(paciente: String, novosDados: String)
        -temPermissao(): boolean
    }

    SistemaSaude <|.. SistemaSaudeReal
    SistemaSaude <|.. ProxySistemaSaude
```

##Estrutura GOF
![image](https://github.com/user-attachments/assets/778e1992-85ca-4506-a5ee-cb1df4c2cb6b)

## Exemplo do Código
```java
interface SistemaSaude {
    void visualizarDados(String paciente);
    void editarDados(String paciente, String novosDados);
}

class SistemaSaudeReal implements SistemaSaude {
    @Override
    public void visualizarDados(String paciente) {
        System.out.println("Visualizando dados do paciente: " + paciente);
    }

    @Override
    public void editarDados(String paciente, String novosDados) {
        System.out.println("Editando dados do paciente: " + paciente + " com novos dados: " + novosDados);
    }
}

class ProxySistemaSaude implements SistemaSaude {
    private SistemaSaudeReal sistemaReal; // Referência para o objeto real
    private String usuario; // Usuário atual

    public ProxySistemaSaude(String usuario) {
        this.usuario = usuario;
        this.sistemaReal = new SistemaSaudeReal(); // Cria o objeto real
    }

    @Override
    public void visualizarDados(String paciente) {
        if (temPermissao()) {
            sistemaReal.visualizarDados(paciente);
        } else {
            System.out.println("Acesso negado. Você não tem permissão para visualizar os dados do paciente.");
        }
    }

    @Override
    public void editarDados(String paciente, String novosDados) {
        if (temPermissao()) {
            sistemaReal.editarDados(paciente, novosDados);
        } else {
            System.out.println("Acesso negado. Você não tem permissão para editar os dados do paciente.");
        }
    }

    private boolean temPermissao() {
        return "MEDICO".equals(usuario) || "ENFERMEIRO".equals(usuario);
    }
}

public class ExemploProxySaude {
    public static void main(String[] args) {
        SistemaSaude sistemaMedico = new ProxySistemaSaude("MEDICO");
        SistemaSaude sistemaEnfermeiro = new ProxySistemaSaude("ENFERMEIRO");
        SistemaSaude sistemaUsuarioComum = new ProxySistemaSaude("USUARIO");

        sistemaMedico.visualizarDados("Pedro Silva");

        sistemaEnfermeiro.editarDados("Pedro Silva", "Alergia a dipirona");

        sistemaUsuarioComum.visualizarDados("Pedro Silva");

        sistemaUsuarioComum.editarDados("Pedro Silva", "Novo endereço: Rua 123");
    }
}
```
## Participantes:
### Subject (SistemaSaude)
- Define a interface comum para `RealSubject` (SistemaSaudeReal) e `Proxy` (ProxySistemaSaude).
- Permite que o Proxy seja usado no lugar do RealSubject.
### RealSubject (SistemaSaudeReal)
- Implementa a lógica real de acesso e manipulação dos dados de saúde.
- Só é criado quando necessário (Lazy Loading).
### Proxy (ProxySistemaSaude)
- Controla o acesso ao `RealSubject`.
- Verifica permissões antes de permitir o acesso.
- Adia a criação do `RealSubject` até que seja realmente necessário.
### Cliente (ExemploProxySaude)    
- Interage com o Proxy sem saber da existência do `RealSubject`.
- Testa o comportamento do Proxy com diferentes usuários.

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
``` java
class ObjetoPersistente {
    public void carregar() {
        System.out.println("Objeto persistente carregado.");
    }
}

class SmartReference {
    private ObjetoPersistente objeto;
    private int contagemReferencias;

    public void acessar() {
        if (objeto == null) {
            objeto = new ObjetoPersistente();
            objeto.carregar();
        }
        contagemReferencias++;
        System.out.println("Referência acessada. Total de referências: " + contagemReferencias);
    }

    public void liberar() {
        contagemReferencias--;
        if (contagemReferencias <= 0) {
            objeto = null;
            System.out.println("Objeto liberado da memória.");
        }
    }
}
public class ExemploSmartReference {
    public static void main(String[] args) {
        SmartReference referencia = new SmartReference();
        referencia.acessar();
        referencia.acessar();
        referencia.liberar();
        referencia.liberar();
    }
}
```

## Colaborações:
- Dependendo de sua categoria, o Proxy retransmite pedidos ao RealSubject quando adequado.

## Consequências:
- O padrão Proxy introduz uma referência indireta ao acessar objetos, oferecendo várias funcionalidades conforme o tipo de proxy:
- Proxies remotos ocultam a localização de um objeto em um espaço de endereçamento diferente.
- Proxies virtuais otimizam a criação de objetos sob demanda.
- Proxies de proteção e smart references realizam tarefas de gerenciamento ao acessar objetos.
- O copy-on-write é uma otimização que adia a cópia de objetos grandes até que sejam modificados, evitando custos desnecessários. A cópia real do objeto só ocorre quando uma alteração é solicitada. Para isso, o objeto deve ter contagem de referências. Copiar o proxy apenas aumenta essa contagem. Quando a contagem chega a zero, o objeto é removido. Essa abordagem reduz significativamente os custos computacionais ao lidar com objetos pesados.

## Implementação:
O padrão Proxy pode explorar as seguintes características:
1. **Controle de Acesso:** O Proxy verifica se o usuário tem permissão para acessar operações, garantindo segurança em dados sensíveis.
2. **Delegação de Chamadas:** O Proxy delega chamadas ao RealSubject, separando a lógica de controle de acesso da lógica de negócios.
3. **Verificação de Permissões:** O Proxy checa se o usuário é médico ou enfermeiro. Se não, o acesso é negado.
4. **Interface Comum:** Proxy e RealSubject implementam a mesma interface, permitindo interação intercambiável.
5. **Instanciação Sob Demanda:** O Proxy cria o RealSubject apenas quando necessário, otimizando recursos.
6. **Encapsulamento:** O Proxy encapsula a lógica de controle de acesso, facilitando a manutenção e a modificação das regras.

## Usos conhecidos:
**- Sistemas Distribuídos:** Proxies atuam como intermediários entre clientes e objetos remotos, simplificando a comunicação.

**- Proxies Virtuais:** Criam objetos sob demanda, economizando recursos ao evitar a criação de instâncias pesadas até que sejam necessárias.

**- Controle de Acesso:** Proxies de proteção garantem que apenas usuários autorizados possam acessar recursos sensíveis.

## Padrões relacionados:
**- Decorator:** Embora similar ao Proxy, o Decorator adiciona funcionalidades sem restringir acesso.

**- Adapter:** Converte interfaces sem atuar como intermediário.

## Conclusão
O padrão Proxy é uma ferramenta que oferece flexibilidade e controle em sistemas de software. Ao permitir que um objeto atue como um substituto para outro, ele facilita o gerenciamento de acesso, otimiza o uso de recursos e promove um design modular. Com suas diversas aplicações, desde Proxies Remotos até Proxies de Proteção e Referências Inteligentes. 

## Referências
GAMMA, Erich; HELM, Richard; JOHNSON, Ralph; VLISSIDES, John. Padrões de projeto: soluções reutilizáveis de software orientado a objetos. 1. ed. Porto Alegre: Bookman, 2000.

