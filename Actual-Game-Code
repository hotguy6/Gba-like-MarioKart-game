import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.AffineTransform;
import java.awt.geom.Path2D;
import java.util.ArrayList;
import java.util.Random;
import java.io.FileWriter;
import java.io.PrintWriter;
import java.io.IOException;

public class GbaKartLite extends JPanel implements ActionListener, KeyListener {

    // --- NEW: PLAYER NAME STORAGE ---
    private String currentPlayerName;

    // --- SETTINGS ---
    private static final int SCREEN_W = 640; 
    private static final int SCREEN_H = 480;
    private static final int FPS = 30; 
    
    // --- GAME CONSTANTS ---
    private static final int MAP_SIZE = 2000;
    
    // --- STATE MANAGEMENT ---
    private enum State { CHAR_SELECT, MAP_SELECT, RACING, FINISHED }
    private State currentState = State.CHAR_SELECT;

    // --- GAME LOGIC ---
    private Timer gameLoop;
    private long startTime;
    private long finalTime;
    private int coinsCollected = 0;
    private boolean passedCheckpoint = false;
    
    // --- PLAYER PHYSICS ---
    private double x = 0, y = 0;
    private double prevX = 0, prevY = 0; 
    private double speed = 0;
    private double angle = 0; 
    
    // Stats (Set by Racer selection)
    private double statMaxSpeed = 15.0; 
    private double statTurnSpeed = 6.0;
    private double statAcceleration = 0.5;
    
    // Boost / Drift Logic
    private boolean isDrifting = false;
    private int driftChargeTime = 0;
    private double boostSpeed = 0; 
    
    // --- VISUALS ---
    private double wheelAngle = 0; 

    // --- INPUT ---
    private boolean up, down, left, right, space;

    // --- RACER DATA & STATS ---
    static class RacerStats {
        String name;
        Color color;
        double weight; 
        
        RacerStats(String n, Color c, double w) {
            this.name = n; this.color = c; this.weight = w;
        }
    }

    private final RacerStats[] racers = {
        new RacerStats("Mario", Color.RED, 2.0),        
        new RacerStats("Luigi", Color.GREEN, 2.0),      
        new RacerStats("Peach", Color.PINK, 1.5),       
        new RacerStats("Wario", Color.YELLOW, 2.8),     
        new RacerStats("DK", new Color(139,69,19), 3.0), 
        new RacerStats("Toad", Color.WHITE, 1.0),       
        new RacerStats("Yoshi", Color.BLUE, 1.5),       
        new RacerStats("Bowser", Color.BLACK, 3.0)      
    };
    private int selectedRacerIndex = 0;

    // --- MAP DATA ---
    private int selectedMapIndex = 0;
    private final String[] mapNames = { "Peach Circuit", "Shy Guy Beach", "Bowser Castle" };
    private final Color[] mapGroundColors = { new Color(34, 139, 34), new Color(238, 214, 175), new Color(40, 10, 10) };
    
    private ArrayList<Point> coins = new ArrayList<>();
    
    // --- DECORATION DATA ---
    static class Decoration {
        int x, y;
        int type; 
        Color color;
        int size;
        
        Decoration(int x, int y, int type, Color c, int s) {
            this.x = x; this.y = y; this.type = type; this.color = c; this.size = s;
        }
    }
    private ArrayList<Decoration> decorations = new ArrayList<>();

    // --- TRACK DEFINITIONS ---
    private Shape outerTrackShape;
    private Shape innerTrackShape;
    private Rectangle startLineRect; 
    private Rectangle finishLineRect; 
    private Rectangle checkpointRect; 

    // --- CONSTRUCTOR UPDATED TO ACCEPT NAME ---
    public GbaKartLite(String playerName) {
        this.currentPlayerName = playerName;

        this.setPreferredSize(new Dimension(SCREEN_W, SCREEN_H));
        this.setBackground(Color.BLACK);
        this.setFocusable(true); 
        this.addKeyListener(this);

        initTrack(); 

        gameLoop = new Timer(1000 / FPS, this);
        gameLoop.start();
    }
    
    // Ensure we get focus when added to the frame
    @Override
    public void addNotify() {
        super.addNotify();
        this.requestFocusInWindow();
    }

    private void initTrack() {
        Path2D outer = new Path2D.Double();
        Path2D inner = new Path2D.Double();

        // Defaults
        startLineRect = new Rectangle(1100, 1500, 20, 400);
        finishLineRect = new Rectangle(1200, 1500, 20, 400);
        checkpointRect = new Rectangle(800, 100, 400, 400); 

        if (selectedMapIndex == 0) {
            // MAP 1: PEACH CIRCUIT
            outer.append(new Rectangle(100, 100, 1800, 1800), false);
            inner.append(new Rectangle(500, 500, 1000, 1000), false);
        } else if (selectedMapIndex == 1) {
            // MAP 2: SHY GUY BEACH
            outer.append(new java.awt.geom.Ellipse2D.Double(100, 100, 1800, 1800), false);
            inner.append(new java.awt.geom.Ellipse2D.Double(600, 600, 800, 800), false);
            startLineRect = new Rectangle(1000, 1400, 20, 400);
            finishLineRect = new Rectangle(1100, 1400, 20, 400);
            checkpointRect = new Rectangle(900, 100, 200, 400); 
        } else {
            // MAP 3: BOWSER CASTLE
            outer.moveTo(100, 100); outer.lineTo(1900, 100); outer.lineTo(1900, 1900); outer.lineTo(100, 1900); outer.closePath();
            inner.moveTo(500, 500); inner.lineTo(1500, 500); inner.lineTo(1500, 900); inner.lineTo(900, 900); 
            inner.lineTo(900, 1300); inner.lineTo(1500, 1300); inner.lineTo(1500, 1500); inner.lineTo(500, 1500); inner.closePath();
            startLineRect = new Rectangle(1000, 1500, 20, 400);
            finishLineRect = new Rectangle(1100, 1500, 20, 400);
            checkpointRect = new Rectangle(1000, 100, 200, 400); 
        }
        
        outerTrackShape = outer;
        innerTrackShape = inner;
        spawnDecorations();
    }

    private void spawnDecorations() {
        decorations.clear();
        Random r = new Random();
        int count = 100; 
        
        for(int i=0; i<count; i++) {
            int dx = r.nextInt(MAP_SIZE);
            int dy = r.nextInt(MAP_SIZE);
            
            boolean isOffRoad = innerTrackShape.contains(dx, dy) || !outerTrackShape.contains(dx, dy);
            
            if(!isOffRoad) { i--; continue; } 
            
            int type = 0; 
            Color col = Color.GREEN;
            int size = 40;
            
            if(selectedMapIndex == 0) { // Peach
                type = 0; col = new Color(34, 139, 34); size = 40 + r.nextInt(30);
            } else if (selectedMapIndex == 1) { // Beach
                type = r.nextBoolean() ? 0 : 1; 
                col = (type == 0) ? new Color(154, 205, 50) : Color.GRAY;
                size = 30 + r.nextInt(30);
            } else { // Bowser
                type = r.nextInt(3) == 0 ? 2 : 1; 
                col = (type == 2) ? new Color(60, 60, 60) : new Color(80, 20, 20);
                size = (type == 2) ? 100 : 30;
            }
            decorations.add(new Decoration(dx, dy, type, col, size));
        }
    }

    private void applyRacerStats() {
        RacerStats r = racers[selectedRacerIndex];
        statMaxSpeed = 16.0 + (r.weight * 0.5); 
        statTurnSpeed = 8.0 - r.weight;          
        statAcceleration = 0.6 - (r.weight * 0.1); 
    }

    private void resetRace() {
        applyRacerStats();
        // SAFE SPAWN COORDS FOR EACH MAP
        if (selectedMapIndex == 0) { x = 900; y = 1700; }
        else if (selectedMapIndex == 1) { x = 900; y = 1600; }
        else { x = 800; y = 1700; } 
        
        prevX = x; prevY = y;
        angle = 0; speed = 0; boostSpeed = 0;
        coinsCollected = 0; passedCheckpoint = false;
        isDrifting = false; driftChargeTime = 0;
        
        spawnCoins();
        startTime = System.currentTimeMillis();
    }

    private void spawnCoins() {
        coins.clear();
        Random r = new Random();
        for (int i = 0; i < 15; i++) {
            int cx, cy;
            int attempts = 0;
            do {
                cx = r.nextInt(MAP_SIZE);
                cy = r.nextInt(MAP_SIZE);
                attempts++;
            } while (!isValidTrackPosition(cx, cy) && attempts < 100); 
            if(attempts < 100) coins.add(new Point(cx, cy));
        }
    }

    private boolean isValidTrackPosition(double tx, double ty) {
        return outerTrackShape.contains(tx, ty) && !innerTrackShape.contains(tx, ty);
    }

    // --- NEW: SAVE TO FILE METHOD ---
    private void saveScoreToFile() {
        try {
            // Append to file (true)
            FileWriter fw = new FileWriter("RaceResults.txt", true);
            PrintWriter printer = new PrintWriter(fw);
            
            String mapName = mapNames[selectedMapIndex];
            String timeStr = formatTime(finalTime);
            
            // Format: "User: [Name] | Map: [Map] | Time: [00:00:00]"
            printer.println("User: " + currentPlayerName + " | Map: " + mapName + " | Time: " + timeStr);
            
            printer.close();
            System.out.println("Score saved successfully!");
            
        } catch (IOException e) {
            System.out.println("Error saving score: " + e.getMessage());
        }
    }

    @Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2d = (Graphics2D) g; 
        
        g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

        switch (currentState) {
            case CHAR_SELECT: drawCharSelect(g2d); break;
            case MAP_SELECT: drawMapSelect(g2d); break;
            case RACING: drawRace(g2d); drawUI(g2d); break;
            case FINISHED: drawRace(g2d); drawFinishedScreen(g2d); break;
        }
    }

    private void drawCharSelect(Graphics g) {
        g.setColor(Color.DARK_GRAY);
        g.fillRect(0, 0, SCREEN_W, SCREEN_H);
        
        // Show Player Name
        g.setColor(Color.CYAN);
        g.setFont(new Font("Arial", Font.BOLD, 20));
        g.drawString("Player: " + currentPlayerName, 20, 30);
        
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("SELECT RACER", SCREEN_W/2 - 150, 60);
        
        for (int i = 0; i < 8; i++) {
            int col = i % 4; int row = i / 4;
            int drawX = 50 + col * (SCREEN_W / 5);
            int drawY = 120 + row * 160;
            
            if (i == selectedRacerIndex) {
                g.setColor(Color.YELLOW);
                g.drawRect(drawX - 5, drawY - 5, 110, 110);
            }
            g.setColor(racers[i].color);
            g.fillRect(drawX, drawY, 100, 100);
            g.setColor(Color.WHITE);
            g.setFont(new Font("Arial", Font.BOLD, 16));
            g.drawString(racers[i].name, drawX + 5, drawY + 120);
            g.setFont(new Font("Arial", Font.PLAIN, 12));
            String wType = (racers[i].weight > 2.0) ? "Heavy" : (racers[i].weight < 1.6) ? "Light" : "Med";
            g.drawString("Type: " + wType, drawX + 5, drawY + 140);
        }
        g.setColor(Color.LIGHT_GRAY);
        g.drawString("Controls: WASD to Drive. SPACE to Drift.", 200, 450);
    }

    private void drawMapSelect(Graphics g) {
        g.setColor(new Color(20, 20, 50));
        g.fillRect(0, 0, SCREEN_W, SCREEN_H);
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("SELECT MAP", SCREEN_W/2 - 120, 100);
        for(int i = 0; i < mapNames.length; i++) {
            int yPos = 250 + (i * 80);
            if(i == selectedMapIndex) {
                g.setColor(Color.YELLOW);
                g.drawString("> " + mapNames[i] + " <", SCREEN_W/2 - 130, yPos);
            } else {
                g.setColor(Color.GRAY);
                g.drawString(mapNames[i], SCREEN_W/2 - 100, yPos);
            }
        }
    }

    private void drawRace(Graphics2D g) {
        AffineTransform old = g.getTransform();
        
        double camX = SCREEN_W / 2.0 - x;
        double camY = SCREEN_H / 2.0 - y;
        g.translate(camX, camY);

        g.setColor(mapGroundColors[selectedMapIndex]); 
        g.fillRect(0, 0, MAP_SIZE, MAP_SIZE);

        drawDecorations(g, false); 

        g.setColor(Color.DARK_GRAY);
        g.fill(outerTrackShape);
        
        g.setColor(mapGroundColors[selectedMapIndex]); 
        g.fill(innerTrackShape); 

        drawDecorations(g, true);

        g.setColor(Color.WHITE);
        g.draw(outerTrackShape);
        g.draw(innerTrackShape);
        g.setColor(Color.RED); g.fill(startLineRect);
        g.setColor(Color.GREEN); g.fill(finishLineRect);
        
        g.setColor(Color.YELLOW);
        for (Point p : coins) {
            g.fillOval(p.x - 15, p.y - 15, 40, 40); 
            g.setColor(Color.ORANGE);
            g.drawOval(p.x - 15, p.y - 15, 40, 40);
        }

        AffineTransform mapBase = g.getTransform();
        g.translate(x, y); 
        g.rotate(Math.toRadians(angle)); 

        if (boostSpeed > 0) {
            g.setColor(new Color(255, 100, 0, 120)); 
            g.fillOval(-60, -20, 40, 10); 
            g.fillOval(-60, 10, 40, 10);  
        }

        Color wheelColor = Color.BLACK;
        if(isDrifting && driftChargeTime > 60) wheelColor = Color.ORANGE; 
        else if(isDrifting && driftChargeTime > 30) wheelColor = Color.CYAN; 
        g.setColor(wheelColor);
        g.fillRect(-15, -22, 15, 6); g.fillRect(-15, 16, 15, 6);    

        AffineTransform carBase = g.getTransform();
        g.translate(17, -19); g.rotate(Math.toRadians(wheelAngle)); g.fillRect(-7, -3, 15, 6); g.setTransform(carBase); 
        g.translate(17, 19); g.rotate(Math.toRadians(wheelAngle)); g.fillRect(-7, -3, 15, 6); g.setTransform(carBase); 

        g.setColor(racers[selectedRacerIndex].color);
        g.fillRoundRect(-30, -20, 60, 40, 10, 10);
        g.setColor(new Color(255, 220, 180));
        g.fillOval(-5, -10, 20, 20);
        
        g.setTransform(old); 
    }

    private void drawDecorations(Graphics2D g, boolean insideInfield) {
        for(Decoration d : decorations) {
            boolean isInside = innerTrackShape.contains(d.x, d.y);
            if (insideInfield && !isInside) continue;
            if (!insideInfield && isInside) continue;
            
            g.setColor(d.color);
            if(d.type == 0) { 
                g.fillOval(d.x - d.size/2, d.y - d.size/2, d.size, d.size);
                g.setColor(d.color.darker());
                g.drawOval(d.x - d.size/2, d.y - d.size/2, d.size, d.size);
            } else if (d.type == 1) { 
                g.fillRect(d.x - d.size/2, d.y - d.size/2, d.size, d.size);
            } else if (d.type == 2) { 
                int[] xPoints = {d.x, d.x - d.size/2, d.x + d.size/2};
                int[] yPoints = {d.y - d.size/2, d.y + d.size/2, d.y + d.size/2};
                g.fillPolygon(xPoints, yPoints, 3);
            }
        }
    }

    private void drawUI(Graphics g) {
        g.setColor(Color.YELLOW);
        g.setFont(new Font("Arial", Font.BOLD, 30));
        g.drawString("COINS: " + coinsCollected, SCREEN_W/2 - 70, 40);

        int mapScale = 15; int mmSize = MAP_SIZE / mapScale; int mmX = 20, mmY = 20;
        g.setColor(new Color(0, 0, 0, 150));
        g.fillRect(mmX, mmY, mmSize, mmSize);
        g.setColor(Color.WHITE);
        g.drawRect(mmX, mmY, mmSize, mmSize);
        g.setColor(racers[selectedRacerIndex].color);
        g.fillRect(mmX + (int)(x / mapScale), mmY + (int)(y / mapScale), 5, 5); 

        if(startTime > 0) {
            long timeElapsed = System.currentTimeMillis() - startTime;
            g.setColor(Color.WHITE);
            g.setFont(new Font("Monospaced", Font.BOLD, 25));
            g.drawString(formatTime(timeElapsed), 20, mmY + mmSize + 30);
        }
        
        if(isDrifting) {
            g.setColor(Color.GRAY);
            g.fillRect(SCREEN_W/2 - 50, SCREEN_H - 50, 100, 10);
            g.setColor(driftChargeTime > 60 ? Color.ORANGE : Color.CYAN);
            int barW = Math.min(100, driftChargeTime * 2);
            g.fillRect(SCREEN_W/2 - 50, SCREEN_H - 50, barW, 10);
            g.setColor(Color.WHITE);
            g.setFont(new Font("Arial", Font.BOLD, 12));
            g.drawString("DRIFT", SCREEN_W/2 - 20, SCREEN_H - 55);
        }
        
        if(!passedCheckpoint && currentState == State.RACING) {
            g.setColor(Color.LIGHT_GRAY);
            g.drawString("Target: Top of Map", 20, mmY + mmSize + 50);
        }
    }
    
    private void drawFinishedScreen(Graphics g) {
        g.setColor(new Color(0,0,0, 150));
        g.fillRect(0, 0, SCREEN_W, SCREEN_H);
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 60));
        g.drawString("FINISHED!", SCREEN_W/2 - 150, 200);
        g.setColor(Color.YELLOW);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("TIME: " + formatTime(finalTime), SCREEN_W/2 - 120, 300);
        g.drawString("COINS: " + coinsCollected, SCREEN_W/2 - 100, 360);
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.PLAIN, 20));
        g.drawString("Score Saved to RaceResults.txt", SCREEN_W/2 - 130, 420);
        g.drawString("Press ENTER to return to Menu", SCREEN_W/2 - 140, 450);
    }
    
    private String formatTime(long millisInput) {
        long seconds = (millisInput / 1000) % 60;
        long minutes = (millisInput / 1000) / 60;
        long ms = (millisInput / 10) % 100;
        return String.format("%02d:%02d:%02d", minutes, seconds, ms);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (currentState == State.RACING) updatePhysics();
        repaint();
    }

    private void updatePhysics() {
        prevX = x; 
        prevY = y;

        if (up) speed += statAcceleration;
        if (down) speed -= statAcceleration;
        speed *= 0.96; 

        if (space && Math.abs(speed) > 3.0) {
            isDrifting = true; driftChargeTime++; 
        } else {
            if (isDrifting) {
                if (driftChargeTime > 60) boostSpeed = 12.0; 
                else if (driftChargeTime > 30) boostSpeed = 6.0; 
            }
            isDrifting = false; driftChargeTime = 0;
        }

        if (boostSpeed > 0) {
            speed = statMaxSpeed + boostSpeed; 
            boostSpeed *= 0.92; 
            if (boostSpeed < 0.1) boostSpeed = 0;
        } else {
            if (speed > statMaxSpeed) speed = statMaxSpeed;
            if (speed < -5.0) speed = -5.0; 
        }

        double targetWheelAngle = 0;
        if (left) targetWheelAngle = -35;
        if (right) targetWheelAngle = 35;
        if (wheelAngle < targetWheelAngle) wheelAngle += 10;
        if (wheelAngle > targetWheelAngle) wheelAngle -= 10;

        if (Math.abs(speed) > 0.1) {
            double turnDir = Math.signum(speed); 
            double turnAmt = statTurnSpeed;
            if (isDrifting) turnAmt *= 1.3;
            if (left) angle -= turnAmt * turnDir; 
            if (right) angle += turnAmt * turnDir;
        }

        double moveAngle = angle;
        if (isDrifting) {
            if (left) moveAngle += 15; if (right) moveAngle -= 15;
        }

        double nextX = x + Math.cos(Math.toRadians(moveAngle)) * speed;
        double nextY = y + Math.sin(Math.toRadians(moveAngle)) * speed;

        if (isValidTrackPosition(nextX, nextY)) { 
            x = nextX; 
            y = nextY; 
        } else { 
            speed = -speed * 0.5; 
            x = prevX; 
            y = prevY;
        }

        if (checkpointRect.contains(x, y)) passedCheckpoint = true;
        
        // --- MODIFIED FINISH LOGIC ---
        if (finishLineRect.contains(x, y) && passedCheckpoint) {
             finalTime = System.currentTimeMillis() - startTime; 
             currentState = State.FINISHED;
             saveScoreToFile(); // Auto-save!
        }
        
        Rectangle playerRect = new Rectangle((int)x - 20, (int)y - 20, 40, 40);
        for (int i = 0; i < coins.size(); i++) {
            Rectangle coinRect = new Rectangle(coins.get(i).x - 15, coins.get(i).y - 15, 40, 40);
            if (playerRect.intersects(coinRect)) { coins.remove(i); coinsCollected++; break; }
        }
    }

    @Override
    public void keyPressed(KeyEvent e) {
        int key = e.getKeyCode();
        if (currentState == State.CHAR_SELECT) {
            if (key == KeyEvent.VK_RIGHT) selectedRacerIndex = (selectedRacerIndex + 1) % 8;
            if (key == KeyEvent.VK_LEFT) selectedRacerIndex = (selectedRacerIndex - 1 + 8) % 8;
            if (key == KeyEvent.VK_DOWN) selectedRacerIndex = (selectedRacerIndex + 4) % 8;
            if (key == KeyEvent.VK_UP) selectedRacerIndex = (selectedRacerIndex - 4 + 8) % 8;
            if (key == KeyEvent.VK_ENTER) { initTrack(); currentState = State.MAP_SELECT; }
        } else if (currentState == State.MAP_SELECT) {
            if (key == KeyEvent.VK_DOWN) selectedMapIndex = (selectedMapIndex + 1) % mapNames.length;
            if (key == KeyEvent.VK_UP) selectedMapIndex = (selectedMapIndex - 1 + mapNames.length) % mapNames.length;
            if (key == KeyEvent.VK_ENTER) { initTrack(); currentState = State.RACING; resetRace(); }
        } else if (currentState == State.RACING) {
            if (key == KeyEvent.VK_W) up = true;
            if (key == KeyEvent.VK_S) down = true;
            if (key == KeyEvent.VK_A) left = true;
            if (key == KeyEvent.VK_D) right = true;
            if (key == KeyEvent.VK_SPACE) space = true;
        } else if (currentState == State.FINISHED) {
            if (key == KeyEvent.VK_ENTER) currentState = State.CHAR_SELECT; 
        }
    }

    @Override
    public void keyReleased(KeyEvent e) {
        if (currentState == State.RACING) {
            int key = e.getKeyCode();
            if (key == KeyEvent.VK_W) up = false;
            if (key == KeyEvent.VK_S) down = false;
            if (key == KeyEvent.VK_A) left = false;
            if (key == KeyEvent.VK_D) right = false;
            if (key == KeyEvent.VK_SPACE) space = false;
        }
    }

    @Override
    public void keyTyped(KeyEvent e) {}

    // Main method for testing without the Login Page
    public static void main(String[] args) {
        JFrame frame = new JFrame("GbaKart Lite - Debug");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setResizable(false);
        frame.add(new GbaKartLite("DebugUser")); // Default name for testing
        frame.pack();
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
