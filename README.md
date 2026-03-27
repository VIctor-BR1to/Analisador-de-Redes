

import java.util.Scanner;

public class AnalisadorRedes {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.print("Digite o domínio para análise: ");
        String dominio = scanner.nextLine();
        
        ResolutorDns dns = new ResolutorDns();
        TestadorPortas testador = new TestadorPortas();
        RequisitorHTTP requisitor = new RequisitorHTTP();     
        
        System.out.println("\n=== PARTE 1 - RESOLUÇÃO DNS ===");
        dns.resolver(dominio);
        
        System.out.println("\n=== PARTE 2 - TESTE DE PORTAS TCP ===");
        testador.testar(dominio);
        
        System.out.println("\n=== PARTE 3 - REQUISIÇÃO HTTP MANUAL ===");
        requisitor.fazerRequisicao(dominio);
        
        scanner.close();
    }
}

------------------------------------


import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.InetSocketAddress;
import java.net.Socket;

public class RequisitorHTTP {
    
    private final int PORTA_HTTP = 80;                  
    private final int TIMEOUT = 5000;                   
    
    public void fazerRequisicao(String dominio) {
        try (Socket socket = new Socket()) {  //                                
            socket.connect(new InetSocketAddress(dominio, PORTA_HTTP), TIMEOUT);
            
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            enviarRequisicao(out, dominio);                                         
            
            analisarResposta(socket.getInputStream(), dominio);
            
        } catch (IOException e) {                                            
            System.out.println("Erro na requisição HTTP: " + e.getMessage());
        }
    }
    
    private void enviarRequisicao(PrintWriter out, String dominio) {    
        out.println("GET / HTTP/1.1");
        out.println("Host: " + dominio);                            
        out.println("User-Agent: AnalisadorRede/1.0");
        out.println("Connection: close");
        out.println();
    }
    
    private void analisarResposta(java.io.InputStream input, String dominio) throws IOException {
        BufferedReader in = new BufferedReader(new InputStreamReader(input));
        String linha;
        String status = "";
        String server = "";    
        String contentType = "";
        String contentLength = "";
        String date = "";
        
        boolean headersFim = false;
        int tamanhoResposta = 0;
        
        while ((linha = in.readLine()) != null) {  
            if (!headersFim) {
                tamanhoResposta += linha.getBytes().length + 2;
                
                if (status.isEmpty() && linha.startsWith("HTTP/")) {
                    String[] partes = linha.split(" ");
                    if (partes.length > 1) status = partes[1];
                }                                                   
                
                if (linha.startsWith("Server:")) {
                    server = linha.substring(7).trim();
                } else if (linha.startsWith("Content-Type:")) {
                    contentType = linha.substring(12).trim();
                } else if (linha.startsWith("Content-Length:")) {
                    contentLength = linha.substring(14).trim();
                } else if (linha.startsWith("Date:")) {
                    date = linha.substring(5).trim();
                } else if (linha.isEmpty()) {
                    headersFim = true;
                }
            }
        }
        
       
        System.out.println("HTTP Status: " + status);
        System.out.println("Servidor: " + (server.isEmpty() ? "Não encontrado" : server));
        System.out.println("Content-Length: " + (contentLength.isEmpty() ? "Não encontrado" : contentLength));
      
        System.out.println("\n=== PARTE 4 - ANÁLISE DE CABEÇALHOS HTTP ===");
        System.out.println("Server: " + (server.isEmpty() ? "Não encontrado" : server));
        System.out.println("Content-Type: " + (contentType.isEmpty() ? "Não encontrado" : contentType));
        System.out.println("Content-Length: " + (contentLength.isEmpty() ? "Não encontrado" : contentLength));
        System.out.println("Date: " + (date.isEmpty() ? "Não encontrado" : date));
    }
}

------------------------


import java.net.Inet4Address; // Usado para representar um endereço IPv4.
import java.net.InetAddress; //  Classe usada para trabalhar com endereços IP e DNS.
import java.time.Duration; //    Serve para calcular tempo entre dois momentos.
import java.time.Instant; //      Representa um momento exato no tempo.

public class ResolutorDns {
    
    public void resolver(String dominio) {
        try {
            Instant inicio = Instant.now();
            InetAddress[] enderecos = InetAddress.getAllByName(dominio);
            Instant fim = Instant.now();
            long tempo = Duration.between(inicio, fim).toMillis();
            
            for (InetAddress endereco : enderecos) {
                String tipo = endereco instanceof Inet4Address ? "A" : "AAAA";
                System.out.printf("Domínio: %s%n", dominio);  //     
                System.out.printf("IP: %s%n", endereco.getHostAddress());
                System.out.printf("Tipo: %s%n", tipo);
                System.out.printf("Tempo DNS: %d ms%n", tempo);
            }
        } catch (Exception e) {
            System.out.println("Erro ao resolver DNS: " + e.getMessage());
        }
    }
}
----------------


import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;

public class TestadorPortas {
    
    private final int[] PORTAS = {80, 443, 21};
    private final int TIMEOUT = 3000;
    
    public void testar(String dominio) {
        for (int porta : PORTAS) {               
            testarPorta(dominio, porta);
        }
    }
    
    private void testarPorta(String dominio, int porta) {
        try (Socket socket = new Socket()) {               
            socket.connect(new InetSocketAddress(dominio, porta), TIMEOUT);
            System.out.printf("Porta %d: aberta%n", porta);
        } catch (IOException e) {
            System.out.printf("Porta %d: fechada%n", porta);
        }
    }
}
