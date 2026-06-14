import javax.swing.*;
import javax.swing.border.*;
import java.awt.*;
import java.awt.event.*;
import java.util.*;
import java.util.prefs.Preferences;

public class WordScrambler extends JFrame {

    // ---- enums ----

    enum Screen { HOME, PLAYING, GAMEOVER }
    enum Difficulty { EASY, MEDIUM, HARD }

    // ---- colours ----

    private static final Color ACCENT        = new Color(0x6C63FF);
    private static final Color ACCENT_BRIGHT = new Color(0x9B94FF);
    private static final Color TEXT_PRIMARY  = new Color(0xF0F0F0);
    private static final Color TEXT_MUTED    = new Color(0x9E9E9E);
    private static final Color BG_DARK       = new Color(0x1A1A2E);

    // ---- word lists ----

    private static final List<String> EASY_WORDS = Arrays.asList(
        "apple","beach","clock","dance","flame","grape","house","juice",
        "knife","lemon","magic","night","ocean","pizza","queen","river",
        "smile","table","tiger","cloud","bread","chair","eagle","field",
        "frost","globe","heart","horse","jewel","light","music","olive",
        "plant","prism","robin","sheep","sleep","smoke","snake","solar",
        "spark","spoon","sport","stack","stage","stamp","stand","steam",
        "steel","stone","storm","story","stove","sugar","sweet","sword",
        "tower","trace","track","train","trend","trial","tribe","trick",
        "truck","trunk","trust","truth","ultra","union","unity","upper",
        "upset","urban","value","video","viola","viper","visit","vista",
        "vital","vivid","vocal","vogue","voice","voter","twirl","twist","blaze"
    );

    private static final List<String> MEDIUM_WORDS = Arrays.asList(
        "balance","captain","crystal","dolphin","eclipse","fantasy",
        "freedom","gallery","harmony","iceberg","journey","kingdom",
        "lantern","mystery","network","outside","perfect","quarter",
        "rainbow","shelter","texture","thunder","village","whisper",
        "admiral","blanket","cabinet","ceiling","comfort","counter",
        "courage","culture","declare","delight","diamond","discuss",
        "display","distant","dresser","element","emperor","embrace",
        "eternal","examine","express","extreme","factory","failure",
        "feature","feeling","fiction","finance","finding","fitness",
        "flatten","flavour","foreign","fortune","furnish","general",
        "genuine","gesture","glitter","gravity","grocery","horizon",
        "hundred","improve","include","involve","justice","kitchen",
        "lasting","liberal","liberty","library","license","logical",
        "massive","mention","message","mineral","miracle","morning",
        "natural","package","painful","passion","pattern","penalty",
        "popular","private","problem","process","program","project",
        "promise","protect","provide","publish","purpose","qualify",
        "reading","reality","receive","recover","reflect","release",
        "require","respect","respond","restore","revenue","routine",
        "science","section","serious","several","shallow","similar",
        "skilled","society","student","success","surface","thought",
        "typical","usually"
    );

    private static final List<String> HARD_WORDS = Arrays.asList(
        "absolute","accident","accuracy","addition","alphabet","ambition",
        "analysis","balanced","behavior","boundary","calendar","campaign",
        "capacity","carnival","casualty","category","ceremony","champion",
        "chemical","commerce","complete","consider","constant","continue",
        "corridor","cylinder","deadline","decision","delivery","describe",
        "disaster","document","dominant","duration","dynamics","earnings",
        "educated","election","elegance","emphasis","employee","enormous",
        "entrance","equality","evaluate","everyday","evidence","exchange",
        "exercise","external","feedback","festival","finalize","forecast",
        "frequent","function","generate","gorgeous","graduate","guidance",
        "hardware","heritage","identify","ignorant","imagined","increase",
        "industry","instance","interest","interior","invasion","lavender",
        "learning","maximize","medicine","medieval","minimize","minority",
        "misplace","movement","multiple","national","negative","numerous",
        "obstacle","official","organize","outreach","paradise","parallel",
        "patience","peaceful","peculiar","personal","platinum","positive",
        "previous","probable","proceeds","products","progress","province",
        "question","randomly","reaction","recovery","redirect","regulate",
        "relation","relative","relevant","remember","research","response",
        "scenario","schedule","separate","shortage","solution","spectrum",
        "standard","strategy","strength","template","together","tomorrow",
        "transfer","ultimate","umbrella","variance","velocity","vertical",
        "violence","wildfire","wireless","workshop","yearning","sparkles"
    );

    // ---- game state ----

    private Screen     currentScreen = Screen.HOME;
    private Difficulty difficulty    = Difficulty.EASY;
    private int        score         = 0;
    private int        wordsSolved   = 0;
    private int        totalAttempts = 0;
    private int        timeLeft      = 120;
    private String     currentWord   = "";
    private boolean    hintUsed      = false;
    private final Set<String> usedWords = new HashSet<>();
    private final Random      random    = new Random();

    // ---- persistence ----

    private final Preferences prefs = Preferences.userNodeForPackage(WordScrambler.class);

    // ---- UI ----

    private final CardLayout cardLayout = new CardLayout();
    private final JPanel     rootPanel  = new JPanel(cardLayout);

    private JComboBox<String> difficultyCombo;
    private JLabel            homeHighScoreLabel;

    private JLabel     timerLabel;
    private JLabel     scoreLabel;
    private JLabel     scrambledLabel;
    private JLabel     feedbackLabel;
    private JTextField answerField;
    private JButton    hintButton;
    private JButton    skipButton;

    private JLabel finalScoreLabel;
    private JLabel wordsSolvedLabel;
    private JLabel accuracyLabel;
    private JLabel highScoreLabel;

    private Timer countdownTimer;

    // ================================================================== main

    public static void main(String[] args) {
        System.setProperty("awt.useSystemAAFontSettings", "on");
        System.setProperty("swing.aatext", "true");
        SwingUtilities.invokeLater(() -> new WordScrambler().setVisible(true));
    }

    // ============================================================ constructor

    public WordScrambler() {
        setTitle("WordScrambler");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(480, 640);
        setMinimumSize(new Dimension(400, 560));
        setLocationRelativeTo(null);
        setBackground(BG_DARK);

        buildHomePanel();
        buildGamePanel();
        buildGameOverPanel();

        add(rootPanel);
        showScreen(Screen.HOME);
    }

    // ============================================================= screen nav

    private void showScreen(Screen screen) {
        currentScreen = screen;
        cardLayout.show(rootPanel, screen.name());
        if (screen == Screen.PLAYING) answerField.requestFocusInWindow();
    }

    // ================================================================ HOME

    private void buildHomePanel() {
        JPanel dark = darkPanel();
        dark.setLayout(new BoxLayout(dark, BoxLayout.Y_AXIS));

        JPanel card = cardPanel();
        card.setLayout(new BoxLayout(card, BoxLayout.Y_AXIS));
        card.setMaximumSize(new Dimension(360, Integer.MAX_VALUE));

        card.add(Box.createVerticalStrut(32));
        card.add(centeredLabel("WORD SCRAMBLER", 32, Font.BOLD, TEXT_PRIMARY));
        card.add(Box.createVerticalStrut(8));
        card.add(centeredLabel("Unscramble words before time runs out.", 14, Font.PLAIN, TEXT_MUTED));
        card.add(Box.createVerticalStrut(32));

        difficultyCombo = styleCombo(new String[]{
            "Easy (4-5 letters)",
            "Medium (6-7 letters)",
            "Hard (8-10 letters)"
        });
        card.add(difficultyCombo);
        card.add(Box.createVerticalStrut(16));

        homeHighScoreLabel = makeLabel("Best: 0", 14, Font.PLAIN, TEXT_MUTED);
        homeHighScoreLabel.setAlignmentX(Component.CENTER_ALIGNMENT);
        card.add(homeHighScoreLabel);
        card.add(Box.createVerticalStrut(24));

        JButton playBtn = accentButton("PLAY");
        playBtn.addActionListener(e -> {
            difficulty = Difficulty.values()[difficultyCombo.getSelectedIndex()];
            startGame();
        });
        card.add(playBtn);
        card.add(Box.createVerticalStrut(32));

        dark.add(Box.createVerticalGlue());
        dark.add(card);
        dark.add(Box.createVerticalGlue());

        updateHomeHighScore();
        rootPanel.add(dark, Screen.HOME.name());
    }

    private void updateHomeHighScore() {
        int best = prefs.getInt("best_" + difficulty.name(), 0);
        homeHighScoreLabel.setText("Best: " + best);
    }

    // =============================================================== PLAYING

    private void buildGamePanel() {
        JPanel dark = darkPanel();
        dark.setLayout(new BorderLayout(12, 12));

        // top bar
        JPanel topBar = new JPanel(new BorderLayout());
        topBar.setOpaque(false);
        timerLabel = makeLabel("2:00", 18, Font.BOLD, TEXT_PRIMARY);
        scoreLabel = makeLabel("Score: 0", 18, Font.BOLD, TEXT_PRIMARY);
        topBar.add(timerLabel, BorderLayout.WEST);
        topBar.add(scoreLabel, BorderLayout.EAST);

        // centre
        JPanel centre = new JPanel();
        centre.setOpaque(false);
        centre.setLayout(new BoxLayout(centre, BoxLayout.Y_AXIS));

        centre.add(centeredLabel("UNSCRAMBLE THIS WORD", 13, Font.BOLD, TEXT_MUTED));
        centre.add(Box.createVerticalStrut(16));

        scrambledLabel = new JLabel("");
        scrambledLabel.setFont(new Font("Monospaced", Font.BOLD, 42));
        scrambledLabel.setForeground(TEXT_PRIMARY);
        scrambledLabel.setAlignmentX(Component.CENTER_ALIGNMENT);
        centre.add(scrambledLabel);
        centre.add(Box.createVerticalStrut(24));

        feedbackLabel = makeLabel("", 14, Font.PLAIN, TEXT_MUTED);
        feedbackLabel.setAlignmentX(Component.CENTER_ALIGNMENT);
        centre.add(feedbackLabel);
        centre.add(Box.createVerticalStrut(16));

        answerField = new JTextField();
        answerField.setForeground(TEXT_PRIMARY);
        answerField.setCaretColor(TEXT_PRIMARY);
        answerField.setBackground(new Color(0x2A2A4A));
        answerField.setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(ACCENT, 2),
            BorderFactory.createEmptyBorder(8, 12, 8, 12)
        ));
        answerField.setHorizontalAlignment(JTextField.CENTER);
        answerField.setFont(new Font("SansSerif", Font.PLAIN, 20));
        answerField.setPreferredSize(new Dimension(280, 48));
        answerField.setMaximumSize(new Dimension(320, 48));
        answerField.addActionListener(e -> submitAnswer());
        centre.add(answerField);
        centre.add(Box.createVerticalStrut(20));

        JButton submitBtn = accentButton("SUBMIT");
        submitBtn.addActionListener(e -> submitAnswer());
        centre.add(submitBtn);
        centre.add(Box.createVerticalStrut(12));

        JPanel auxRow = new JPanel();
        auxRow.setOpaque(false);
        hintButton = ghostButton("Hint (-5 pts)");
        hintButton.addActionListener(e -> useHint());
        skipButton = ghostButton("Skip");
        skipButton.addActionListener(e -> skipWord());
        auxRow.add(hintButton);
        auxRow.add(Box.createHorizontalStrut(12));
        auxRow.add(skipButton);
        centre.add(auxRow);

        dark.add(topBar, BorderLayout.NORTH);
        dark.add(centre, BorderLayout.CENTER);
        rootPanel.add(dark, Screen.PLAYING.name());
    }

    // ============================================================= GAME OVER

    private void buildGameOverPanel() {
        JPanel dark = darkPanel();
        dark.setLayout(new BoxLayout(dark, BoxLayout.Y_AXIS));

        JPanel card = cardPanel();
        card.setLayout(new BoxLayout(card, BoxLayout.Y_AXIS));
        card.setMaximumSize(new Dimension(360, Integer.MAX_VALUE));

        card.add(Box.createVerticalStrut(32));
        card.add(centeredLabel("Time's Up!", 32, Font.BOLD, TEXT_PRIMARY));
        card.add(Box.createVerticalStrut(24));

        finalScoreLabel  = makeLabel("", 28, Font.BOLD, ACCENT_BRIGHT);
        wordsSolvedLabel = makeLabel("Words Solved: 0", 16, Font.PLAIN, TEXT_PRIMARY);
        accuracyLabel    = makeLabel("Accuracy: 0%", 16, Font.PLAIN, TEXT_PRIMARY);
        highScoreLabel   = makeLabel("", 14, Font.BOLD, ACCENT_BRIGHT);

        for (JLabel lbl : new JLabel[]{finalScoreLabel, wordsSolvedLabel, accuracyLabel, highScoreLabel}) {
            lbl.setAlignmentX(Component.CENTER_ALIGNMENT);
            card.add(lbl);
            card.add(Box.createVerticalStrut(8));
        }

        card.add(Box.createVerticalStrut(16));

        JButton playAgainBtn = accentButton("Play Again");
        playAgainBtn.addActionListener(e -> { startGame(); showScreen(Screen.PLAYING); });
        card.add(playAgainBtn);
        card.add(Box.createVerticalStrut(12));

        JButton homeBtn = ghostButton("Main Menu");
        homeBtn.addActionListener(e -> { showScreen(Screen.HOME); updateHomeHighScore(); });
        card.add(homeBtn);
        card.add(Box.createVerticalStrut(32));

        dark.add(Box.createVerticalGlue());
        dark.add(card);
        dark.add(Box.createVerticalGlue());

        rootPanel.add(dark, Screen.GAMEOVER.name());
    }

    // ============================================================= game logic

    private void startGame() {
        score = 0; wordsSolved = 0; totalAttempts = 0;
        timeLeft = 120; hintUsed = false;
        usedWords.clear();
        updateScoreLabel();
        updateTimerLabel();
        nextWord();
        startTimer();
        showScreen(Screen.PLAYING);
    }

    private void nextWord() {
        hintUsed = false;
        hintButton.setEnabled(true);
        answerField.setText("");
        feedbackLabel.setText("");

        List<String> available = new ArrayList<>();
        for (String w : getWordList()) if (!usedWords.contains(w)) available.add(w);
        if (available.isEmpty()) { endGame(); return; }

        currentWord = available.get(random.nextInt(available.size()));
        usedWords.add(currentWord);
        scrambledLabel.setText(scramble(currentWord).toUpperCase());
    }

    private String scramble(String word) {
        char[] chars = word.toCharArray();
        do {
            for (int i = chars.length - 1; i > 0; i--) {
                int j = random.nextInt(i + 1);
                char tmp = chars[i]; chars[i] = chars[j]; chars[j] = tmp;
            }
        } while (new String(chars).equals(word) && word.length() > 1);
        return new String(chars);
    }

    private void submitAnswer() {
        String answer = answerField.getText().trim().toLowerCase();
        totalAttempts++;
        if (answer.equals(currentWord)) {
            int pts = currentWord.length() * 10 + (hintUsed ? 0 : 5);
            score += pts;
            wordsSolved++;
            updateScoreLabel();
            showFeedback("✓ +" + pts + " pts!", ACCENT_BRIGHT, 900);
            nextWord();
        } else {
            showFeedback("Try again!", new Color(0xFF6B6B), 700);
            shakeField();
        }
    }

    private void useHint() {
        if (hintUsed) return;
        hintUsed = true;
        score = Math.max(0, score - 5);
        updateScoreLabel();
        hintButton.setEnabled(false);
        showFeedback("Hint: starts with '" + currentWord.charAt(0) + "'  (-5 pts)", TEXT_MUTED, 3000);
    }

    private void skipWord() {
        totalAttempts++;
        showFeedback("Skipped — the word was: " + currentWord, TEXT_MUTED, 1500);
        nextWord();
    }

    private void startTimer() {
        if (countdownTimer != null) countdownTimer.stop();
        countdownTimer = new Timer(1000, e -> {
            timeLeft--;
            updateTimerLabel();
            if (timeLeft <= 0) endGame();
        });
        countdownTimer.setRepeats(true);
        countdownTimer.start();
    }

    private void endGame() {
        if (countdownTimer != null) countdownTimer.stop();
        String key = "best_" + difficulty.name();
        int best = prefs.getInt(key, 0);
        boolean newHigh = score > best;
        if (newHigh) prefs.putInt(key, score);
        int accuracy = totalAttempts == 0 ? 0 : (int) Math.round(100.0 * wordsSolved / totalAttempts);
        finalScoreLabel.setText("Score: " + score);
        wordsSolvedLabel.setText("Words Solved: " + wordsSolved);
        accuracyLabel.setText("Accuracy: " + accuracy + "%");
        highScoreLabel.setText(newHigh ? "New High Score!" : "Best: " + Math.max(score, best));
        showScreen(Screen.GAMEOVER);
    }

    // ============================================================== helpers

    private List<String> getWordList() {
        switch (difficulty) {
            case EASY:   return EASY_WORDS;
            case MEDIUM: return MEDIUM_WORDS;
            default:     return HARD_WORDS;
        }
    }

    private void updateScoreLabel() { scoreLabel.setText("Score: " + score); }

    private void updateTimerLabel() {
        timerLabel.setText(String.format("%d:%02d", timeLeft / 60, timeLeft % 60));
        timerLabel.setForeground(timeLeft <= 10 ? new Color(0xFF6B6B) : TEXT_PRIMARY);
    }

    private void showFeedback(String msg, Color color, int ms) {
        feedbackLabel.setText(msg);
        feedbackLabel.setForeground(color);
        new Timer(ms, e -> feedbackLabel.setText("")) {{ setRepeats(false); start(); }};
    }

    private void shakeField() {
        Point orig = answerField.getLocation();
        int[] offsets = {-8, 8, -6, 6, -4, 4, 0};
        int[] idx = {0};
        Timer t = new Timer(40, null);
        t.addActionListener(e -> {
            answerField.setLocation(orig.x + offsets[idx[0]], orig.y);
            if (++idx[0] >= offsets.length) { answerField.setLocation(orig); t.stop(); }
        });
        t.start();
    }

    // =========================================================== UI factories

    private JPanel darkPanel() {
        return new JPanel() {
            @Override protected void paintComponent(Graphics g) {
                Graphics2D g2 = (Graphics2D) g.create();
                g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
                g2.setColor(BG_DARK);
                g2.fillRect(0, 0, getWidth(), getHeight());
                g2.dispose();
            }
        };
    }

    private JPanel cardPanel() {
        return new JPanel() {
            @Override protected void paintComponent(Graphics g) {
                Graphics2D g2 = (Graphics2D) g.create();
                g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
                g2.setColor(new Color(0x24243E));
                g2.fillRoundRect(0, 0, getWidth(), getHeight(), 24, 24);
                g2.setColor(ACCENT.darker());
                g2.drawRoundRect(0, 0, getWidth() - 1, getHeight() - 1, 24, 24);
                g2.dispose();
            }
        };
    }

    private JButton accentButton(String text) {
        JButton btn = new JButton(text) {
            @Override protected void paintComponent(Graphics g) {
                Graphics2D g2 = (Graphics2D) g.create();
                g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
                g2.setColor(getModel().isPressed() ? ACCENT.darker()
                          : getModel().isRollover() ? ACCENT_BRIGHT : ACCENT);
                g2.fillRoundRect(0, 0, getWidth(), getHeight(), 16, 16);
                g2.dispose();
                super.paintComponent(g);
            }
        };
        btn.setForeground(Color.WHITE);
        btn.setOpaque(false); btn.setContentAreaFilled(false);
        btn.setBorderPainted(false); btn.setFocusPainted(false);
        btn.setFont(new Font("SansSerif", Font.BOLD, 15));
        btn.setMaximumSize(new Dimension(280, 44));
        btn.setAlignmentX(Component.CENTER_ALIGNMENT);
        btn.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        return btn;
    }

    private JButton ghostButton(String text) {
        JButton btn = new JButton(text) {
            @Override protected void paintComponent(Graphics g) {
                Graphics2D g2 = (Graphics2D) g.create();
                g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
                if (getModel().isRollover()) {
                    g2.setColor(new Color(ACCENT.getRed(), ACCENT.getGreen(), ACCENT.getBlue(), 40));
                    g2.fillRoundRect(0, 0, getWidth(), getHeight(), 16, 16);
                }
                g2.setColor(ACCENT);
                g2.drawRoundRect(0, 0, getWidth() - 1, getHeight() - 1, 16, 16);
                g2.dispose();
                super.paintComponent(g);
            }
        };
        btn.setForeground(ACCENT_BRIGHT);
        btn.setOpaque(false); btn.setContentAreaFilled(false);
        btn.setBorderPainted(false); btn.setFocusPainted(false);
        btn.setFont(new Font("SansSerif", Font.PLAIN, 14));
        btn.setMaximumSize(new Dimension(200, 38));
        btn.setAlignmentX(Component.CENTER_ALIGNMENT);
        btn.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        return btn;
    }

    private JComboBox<String> styleCombo(String[] items) {
        JComboBox<String> combo = new JComboBox<>(items);
        combo.setBackground(new Color(0x2A2A4A));
        combo.setForeground(TEXT_PRIMARY);
        combo.setFont(new Font("SansSerif", Font.PLAIN, 14));
        combo.setMaximumSize(new Dimension(280, 38));
        combo.setAlignmentX(Component.CENTER_ALIGNMENT);
        return combo;
    }

    private JLabel makeLabel(String text, int size, int style, Color color) {
        JLabel lbl = new JLabel(text);
        lbl.setFont(new Font("SansSerif", style, size));
        lbl.setForeground(color);
        return lbl;
    }

    private JLabel centeredLabel(String text, int size, int style, Color color) {
        JLabel lbl = makeLabel(text, size, style, color);
        lbl.setAlignmentX(Component.CENTER_ALIGNMENT);
        return lbl;
    }
}# word-scramble
