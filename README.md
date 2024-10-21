import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.ArrayList;
import java.util.Collections;

public class Main {
    private static JFrame frame;
    private static JPanel painelInicial, painelJogo;
    private static ArrayList<ImageIcon> cartas;
    private static JButton[] botoes;
    private static int paresEncontrados = 0;
    private static JLabel contadorMovimentosLabel, pontosLabel, timerLabel, nivelLabel;
    private static int contadorMovimentos = 0;
    private static int totalPares = 8;
    private static int pontos = 0;
    private static int nivel = 1; // Contador de fase
    private static ImageIcon versoCarta = redimensionarImagem(new ImageIcon("imagens/verso_carta.jpg"), 80, 80);
    private static Timer cronometro;
    private static int primeiroClique = -1, segundoClique = -1;
    private static int segundos = 60; // Tempo inicial em segundos para cada nível

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            frame = new JFrame("Jogo da Memória");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(800, 600);
            frame.setLayout(new BorderLayout());

            // Painel inicial
            painelInicial = new JPanel() {
                @Override
                protected void paintComponent(Graphics g) {
                    super.paintComponent(g);
                    ImageIcon background = new ImageIcon("imagens/painel_inicial.jpg");
                    g.drawImage(background.getImage(), 0, 0, getWidth(), getHeight(), null);
                }
            };
            painelInicial.setLayout(new BorderLayout());

            JButton botaoJogar = new JButton("Jogar");
            botaoJogar.setFont(new Font("Arial", Font.PLAIN, 18));
            botaoJogar.setPreferredSize(new Dimension(200, 50));
            botaoJogar.setBackground(Color.yellow);
            botaoJogar.setForeground(Color.BLACK);
            botaoJogar.setFocusPainted(false);
            botaoJogar.setBorderPainted(false);
            painelInicial.add(botaoJogar, BorderLayout.SOUTH);
            frame.add(painelInicial, BorderLayout.CENTER);
            frame.setVisible(true);

            botaoJogar.addActionListener(new ActionListener() {
                @Override
                public void actionPerformed(ActionEvent e) {
                    iniciarJogo();
                }
            });
        });
    }

    private static void iniciarJogo() {
        frame.getContentPane().removeAll();
        painelJogo = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                ImageIcon background = new ImageIcon("imagens/fundo_jogo.jpg");
                g.drawImage(background.getImage(), 0, 0, getWidth(), getHeight(), null);
            }
        };
        painelJogo.setLayout(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(10, 10, 10, 10);

        // Carregar imagens das cartas
        cartas = new ArrayList<>();
        for (int i = 1; i <= totalPares; i++) {
            ImageIcon carta = redimensionarImagem(new ImageIcon("imagens/carta" + i + "a.jpg"), 80, 80);
            cartas.add(carta);
            cartas.add(carta); // Adiciona a imagem duas vezes para formar um par
        }
        Collections.shuffle(cartas);
        botoes = new JButton[totalPares * 2];
        int gridRow = 0;
        int gridCol = 0;
        for (int i = 0; i < totalPares * 2; i++) {
            botoes[i] = new JButton();
            botoes[i].setIcon(versoCarta);
            botoes[i].setPreferredSize(new Dimension(80, 80));
            botoes[i].setBorder(BorderFactory.createEmptyBorder());
            botoes[i].addActionListener(new CartaActionListener(i));
            gbc.gridx = gridCol;
            gbc.gridy = gridRow;
            painelJogo.add(botoes[i], gbc);
            gridCol++;
            if (gridCol == 4) {
                gridCol = 0;
                gridRow++;
            }
        }

        // Adicionar contador de movimentos, pontos, nível e timer centralizados no topo
        contadorMovimentosLabel = new JLabel("Movimentos: 0");
        contadorMovimentosLabel.setFont(new Font("Arial", Font.PLAIN, 24)); // Tamanho do contador de movimentos
        pontosLabel = new JLabel("Pontos: " + pontos);
        pontosLabel.setFont(new Font("Arial", Font.PLAIN, 24)); // Tamanho do contador de pontos
        timerLabel = new JLabel("Tempo: " + segundos);
        timerLabel.setFont(new Font("Arial", Font.PLAIN, 24)); // Tamanho do contador de tempo
        nivelLabel = new JLabel("Fase: " + nivel);
        nivelLabel.setFont(new Font("Arial", Font.PLAIN, 24)); // Tamanho do contador de nível

        JPanel infoPanel = new JPanel();
        infoPanel.setLayout(new GridBagLayout());
        GridBagConstraints infoGbc = new GridBagConstraints();
        infoGbc.insets = new Insets(0, 10, 0, 10);
        infoGbc.gridx = 0;
        infoPanel.add(contadorMovimentosLabel, infoGbc);
        infoGbc.gridx = 1;
        infoPanel.add(pontosLabel, infoGbc);
        infoGbc.gridx = 2;
        infoPanel.add(timerLabel, infoGbc);
        infoGbc.gridx = 3;
        infoPanel.add(nivelLabel, infoGbc);
        frame.add(infoPanel, BorderLayout.NORTH);
        frame.add(painelJogo, BorderLayout.CENTER);
        frame.revalidate();
        frame.repaint();

        // Iniciar cronômetro
        if (cronometro != null) {
            cronometro.stop();
        }
        cronometro = new Timer(1000, new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                if (segundos > 0) {
                    segundos--;
                    timerLabel.setText("Tempo: " + segundos);
                } else {
                    ((Timer) e.getSource()).stop();
                    JOptionPane.showMessageDialog(frame, "Tempo esgotado! Você perdeu!");
                    resetarJogoCompleto();
                }
            }
        });
        cronometro.start();
    }

    private static void resetarJogoCompleto() {
        paresEncontrados = 0;
        contadorMovimentos = 0;
        totalPares = 8;
        pontos = 0;
        nivel = 1;
        segundos = 60; // Reiniciar o tempo para o primeiro nível
        if (cronometro != null) {
            cronometro.stop();
        }
        voltarParaTelaInicial();
    }

    private static void voltarParaTelaInicial() {
        frame.getContentPane().removeAll();
        frame.add(painelInicial, BorderLayout.CENTER);
        frame.revalidate();
        frame.repaint();
    }

    private static ImageIcon redimensionarImagem(ImageIcon imageIcon, int largura, int altura) {
        Image image = imageIcon.getImage();
        Image newimg = image.getScaledInstance(largura, altura, java.awt.Image.SCALE_SMOOTH);
        return new ImageIcon(newimg);
    }

    private static class CartaActionListener implements ActionListener {
        private int index;

        public CartaActionListener(int index) {
            this.index = index;
        }

        @Override
        public void actionPerformed(ActionEvent e) {
            if (primeiroClique == -1) {
                primeiroClique = index;
                botoes[index].setIcon(cartas.get(index));
            } else if (segundoClique == -1 && index != primeiroClique) {
                segundoClique = index;
                botoes[index].setIcon(cartas.get(index));
                contadorMovimentos++;
                contadorMovimentosLabel.setText("Movimentos: " + contadorMovimentos);

                // Verificar se as cartas são iguais
                if (cartas.get(primeiroClique).equals(cartas.get(segundoClique))) {
                    botoes[primeiroClique].setEnabled(false);
                    botoes[segundoClique].setEnabled(false);
                    paresEncontrados++;
                    pontos++; // Incrementar pontos para cada par correto
                    pontosLabel.setText("Pontos: " + pontos);
                    primeiroClique = -1;
                    segundoClique = -1;
                    if (pontos >= 108) {
                        cronometro.stop();
                        JOptionPane.showMessageDialog(frame, "Parabéns! Você venceu o jogo!");
                        resetarJogoCompleto();
                        return;
                    }
                    if (paresEncontrados == totalPares) {
                        cronometro.stop();
                        JOptionPane.showMessageDialog(frame, "Parabéns! Você encontrou todos os pares!");
                        totalPares++; // Aumentar o total de pares para a próxima fase
                        segundos = 60 + (nivel * 20); // Aumentar o tempo para a próxima fase
                        nivel++; // Aumentar o nível
                        paresEncontrados = 0; // Resetar pares encontrados
                        contadorMovimentos = 0; // Resetar movimentos
                        contadorMovimentosLabel.setText("Movimentos: " + contadorMovimentos);
                        pontosLabel.setText("Pontos: " + pontos);
                        nivelLabel.setText("Fase: " + nivel);
                        iniciarJogo(); // Reiniciar o jogo para a próxima fase
                    }
                } else {
                    Timer timer = new Timer(1000, new ActionListener() {
                        @Override
                        public void actionPerformed(ActionEvent e) {
                            botoes[primeiroClique].setIcon(versoCarta);
                            botoes[segundoClique].setIcon(versoCarta);
                            primeiroClique = -1;
                            segundoClique = -1;
                        }
                    });
                    timer.setRepeats(false);
                    timer.start();
                }
            }
        }
    }
}
