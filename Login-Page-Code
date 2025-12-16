import javax.swing.*;
import java.awt.event.*;
import java.util.HashMap;

public class LoginPage implements ActionListener {

    HashMap<String, String> userDatabase = new HashMap<>();

    JFrame frame = new JFrame("Simple Login");
    JTextField userField = new JTextField();
    JPasswordField passField = new JPasswordField();
    JButton btnRegister = new JButton("Register");
    JButton btnLogin = new JButton("Login");
    JLabel msgLabel = new JLabel("");

    public LoginPage() {

        JLabel uLabel = new JLabel("User:");
        uLabel.setBounds(50, 50, 80, 25);
        userField.setBounds(100, 50, 160, 25);

        JLabel pLabel = new JLabel("Pass:");
        pLabel.setBounds(50, 100, 80, 25);
        passField.setBounds(100, 100, 160, 25);

        btnRegister.setBounds(50, 150, 100, 25);
        btnLogin.setBounds(160, 150, 100, 25);
        msgLabel.setBounds(50, 200, 250, 25);


        btnRegister.addActionListener(this);
        btnLogin.addActionListener(this);


        frame.add(uLabel); frame.add(userField);
        frame.add(pLabel); frame.add(passField);
        frame.add(btnRegister); frame.add(btnLogin);
        frame.add(msgLabel);

        frame.setSize(350, 300);
        frame.setLayout(null);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        String user = userField.getText();
        String pass = String.valueOf(passField.getPassword());

        if (e.getSource() == btnRegister) {
            if (user.isEmpty() || pass.isEmpty()) {
                msgLabel.setText("Cannot register empty fields.");
            } else {
                userDatabase.put(user, pass);
                msgLabel.setText("Registered! Now Login.");
            }
        } 
        else if (e.getSource() == btnLogin) {

            if (userDatabase.containsKey(user) && userDatabase.get(user).equals(pass)) {
                frame.dispose(); 
                launchGame(user);
            } else {
                msgLabel.setText("Invalid Username or Password.");
            }
        }
    }

    public void launchGame(String playerName) {
        JFrame gameFrame = new JFrame("GbaKart - Player: " + playerName);
        gameFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        gameFrame.setResizable(false);
        
        GbaKartLite gamePanel = new GbaKartLite(playerName);
        
        gameFrame.add(gamePanel);
        gameFrame.pack();
        gameFrame.setLocationRelativeTo(null);
        gameFrame.setVisible(true);
        gamePanel.requestFocus();
    }

    public static void main(String[] args) {
        new LoginPage();
    }
}
