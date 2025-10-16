import java.util.*;

public class TexasHoldemPersonalityChoice {

    static class Card {
        String rank, suit;
        int value;

        Card(String rank, String suit, int value) {
            this.rank = rank;
            this.suit = suit;
            this.value = value;
        }

        @Override
        public String toString() {
            return rank + " of " + suit;
        }
    }

    static class Deck {
        private final List<Card> cards = new ArrayList<>();
        private final String[] suits = {"Hearts", "Diamonds", "Clubs", "Spades"};
        private final String[] ranks = {"2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"};

        Deck() {
            for (String suit : suits) {
                for (int i = 0; i < ranks.length; i++) {
                    cards.add(new Card(ranks[i], suit, i + 2));
                }
            }
            shuffle();
        }

        void shuffle() {
            Collections.shuffle(cards);
        }

        Card dealCard() {
            return cards.remove(0);
        }
    }

    static class Player {
        String name;
        List<Card> hand = new ArrayList<>();
        boolean folded = false;

        Player(String name) {
            this.name = name;
        }

        void addCard(Card card) {
            hand.add(card);
        }
    }

    enum Personality {
        CAUTIOUS, AGGRESSIVE, BLUFFING
    }

    // --- Poker Evaluation ---
    static int rankHand(List<Card> cards) {
        cards.sort((a, b) -> b.value - a.value);

        boolean flush = isFlush(cards);
        boolean straight = isStraight(cards);
        Map<Integer, Integer> counts = getCounts(cards);

        int four = hasCount(counts, 4);
        int three = hasCount(counts, 3);
        List<Integer> pairs = getPairs(counts);

        if (straight && flush && cards.get(0).value == 14) return 900;
        if (straight && flush) return 800;
        if (four > 0) return 700;
        if (three > 0 && pairs.size() > 0) return 600;
        if (flush) return 500;
        if (straight) return 400;
        if (three > 0) return 300;
        if (pairs.size() == 2) return 200;
        if (pairs.size() == 1) return 100;
        return cards.get(0).value;
    }

    static String getHandName(int rank) {
        if (rank >= 900) return "Royal Flush";
        if (rank >= 800) return "Straight Flush";
        if (rank >= 700) return "Four of a Kind";
        if (rank >= 600) return "Full House";
        if (rank >= 500) return "Flush";
        if (rank >= 400) return "Straight";
        if (rank >= 300) return "Three of a Kind";
        if (rank >= 200) return "Two Pair";
        if (rank >= 100) return "One Pair";
        return "High Card";
    }

    static boolean isFlush(List<Card> cards) {
        Map<String, Integer> suitCounts = new HashMap<>();
        for (Card c : cards) {
            suitCounts.put(c.suit, suitCounts.getOrDefault(c.suit, 0) + 1);
            if (suitCounts.get(c.suit) >= 5)
                return true;
        }
        return false;
    }

    static boolean isStraight(List<Card> cards) {
        Set<Integer> values = new TreeSet<>();
        for (Card c : cards)
            values.add(c.value);
        List<Integer> v = new ArrayList<>(values);
        for (int i = 0; i < v.size() - 4; i++) {
            if (v.get(i + 4) - v.get(i) == 4)
                return true;
        }
        return v.containsAll(Arrays.asList(14, 2, 3, 4, 5));
    }

    static Map<Integer, Integer> getCounts(List<Card> cards) {
        Map<Integer, Integer> counts = new HashMap<>();
        for (Card c : cards)
            counts.put(c.value, counts.getOrDefault(c.value, 0) + 1);
        return counts;
    }

    static int hasCount(Map<Integer, Integer> counts, int count) {
        for (var e : counts.entrySet()) {
            if (e.getValue() == count)
                return e.getKey();
        }
        return 0;
    }

    static List<Integer> getPairs(Map<Integer, Integer> counts) {
        List<Integer> pairs = new ArrayList<>();
        for (var e : counts.entrySet()) {
            if (e.getValue() == 2)
                pairs.add(e.getKey());
        }
        return pairs;
    }

    // --- CPU Decision Logic ---
    static boolean cpuPreFlopFold(Player cpu, Personality personality) {
        Card c1 = cpu.hand.get(0);
        Card c2 = cpu.hand.get(1);
        int avgValue = (c1.value + c2.value) / 2;
        boolean sameSuit = c1.suit.equals(c2.suit);
        boolean connected = Math.abs(c1.value - c2.value) == 1;

        int score = avgValue;
        if (sameSuit) score += 4;
        if (connected) score += 2;
        if (c1.value == c2.value) score += 10;

        System.out.println("\n🤖 My starting hand: " + c1 + " and " + c2);

        int threshold = switch (personality) {
            case CAUTIOUS -> 11;
            case AGGRESSIVE -> 7;
            case BLUFFING -> 5;
        };

        if (score < threshold) {
            System.out.println(cpuDialogue(personality, "preflopFold"));
            return true;
        } else {
            System.out.println(cpuDialogue(personality, "preflopStay"));
            return false;
        }
    }

    static boolean cpuMidRoundFold(Player cpu, List<Card> community, String stage, Personality personality) {
        List<Card> all = new ArrayList<>(cpu.hand);
        all.addAll(community);
        int rank = rankHand(all);
        String handName = getHandName(rank);

        System.out.println("\n🤖 Checking my cards after the " + stage + "...");
        System.out.println("🤖 I have a " + handName + " (rank " + rank + ").");

        int threshold = switch (personality) {
            case CAUTIOUS -> switch (stage) {
                case "flop" -> 120;
                case "turn" -> 160;
                case "river" -> 200;
                default -> 100;
            };
            case AGGRESSIVE -> switch (stage) {
                case "flop" -> 80;
                case "turn" -> 130;
                case "river" -> 170;
                default -> 90;
            };
            case BLUFFING -> 50; // stays almost always
        };

        if (rank < threshold) {
            System.out.println(cpuDialogue(personality, "foldMid"));
            return true;
        } else {
            System.out.println(cpuDialogue(personality, "stayMid"));
            return false;
        }
    }

    // --- Personality Dialogue ---
    static String cpuDialogue(Personality p, String situation) {
        return switch (p) {
            case CAUTIOUS -> switch (situation) {
                case "preflopFold" -> "🤖 These cards are too weak. I’ll fold early.";
                case "preflopStay" -> "🤖 Not bad... I’ll give it a shot.";
                case "foldMid" -> "🤖 Hmm, this isn’t looking safe. I’m out.";
                case "stayMid" -> "🤖 Decent hand — I’ll stay in carefully.";
                default -> "";
            };
            case AGGRESSIVE -> switch (situation) {
                case "preflopFold" -> "🤖 Nah, this hand’s trash... fine, I’ll fold.";
                case "preflopStay" -> "🤖 Let’s go! I like the look of these cards.";
                case "foldMid" -> "🤖 Okay, okay, maybe not my day.";
                case "stayMid" -> "🤖 I’m in — I can feel a big hand coming!";
                default -> "";
            };
            case BLUFFING -> switch (situation) {
                case "preflopFold" -> "🤖 (bluffing) These look fine... I’ll just say I folded to trick you.";
                case "preflopStay" -> "🤖 Easy win. I’ve got this in the bag.";
                case "foldMid" -> "🤖 Heh, you *think* I’m folding... but maybe I’m not.";
                case "stayMid" -> "🤖 Oh yeah, best hand ever. Definitely. Totally.";
                default -> "";
            };
        };
    }

    // --- Main Game ---
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        Deck deck = new Deck();

        System.out.println("♠️ Welcome to Texas Hold’em (CPU Edition)");
        System.out.println("Choose your opponent’s personality:");
        System.out.println("1. 😌 Cautious");
        System.out.println("2. 😈 Aggressive");
        System.out.println("3. 🤥 Bluffing");
        System.out.print("Your choice: ");
        int choice = sc.nextInt();

        Personality personality = switch (choice) {
            case 1 -> Personality.CAUTIOUS;
            case 2 -> Personality.AGGRESSIVE;
            case 3 -> Personality.BLUFFING;
            default -> Personality.CAUTIOUS;
        };

        Player user = new Player("You");
        Player cpu = new Player("CPU");

        for (int i = 0; i < 2; i++) {
            user.addCard(deck.dealCard());
            cpu.addCard(deck.dealCard());
        }

        System.out.println("\n--- Your Hand ---");
        user.hand.forEach(System.out::println);

        if (cpuPreFlopFold(cpu, personality)) {
            System.out.println("\n🤖 folds before the flop!");
            System.out.println("🏆 You win automatically!");
            return;
        }

        System.out.println("\n🤖 stays in for the flop...");

        List<Card> community = new ArrayList<>();

        // Flop
        for (int i = 0; i < 3; i++) community.add(deck.dealCard());
        System.out.println("\n--- FLOP ---");
        community.forEach(System.out::println);

        if (cpuMidRoundFold(cpu, community, "flop", personality)) {
            System.out.println("\n🏆 You win automatically!");
            return;
        }

        // Turn
        community.add(deck.dealCard());
        System.out.println("\n--- TURN ---");
        community.forEach(System.out::println);

        if (cpuMidRoundFold(cpu, community, "turn", personality)) {
            System.out.println("\n🏆 You win automatically!");
            return;
        }

        // River
        community.add(deck.dealCard());
        System.out.println("\n--- RIVER ---");
        community.forEach(System.out::println);

        if (cpuMidRoundFold(cpu, community, "river", personality)) {
            System.out.println("\n🏆 You win automatically!");
            return;
        }

        // Showdown
        List<Card> userCombined = new ArrayList<>(user.hand);
        userCombined.addAll(community);
        int userRank = rankHand(userCombined);

        List<Card> cpuCombined = new ArrayList<>(cpu.hand);
        cpuCombined.addAll(community);
        int cpuRank = rankHand(cpuCombined);

        System.out.println("\n--- CPU Hand ---");
        cpu.hand.forEach(System.out::println);

        System.out.println("\nYour Hand: " + getHandName(userRank));
        System.out.println("CPU Hand: " + getHandName(cpuRank));

        if (userRank > cpuRank) {
            System.out.println("\n🏆 You win! (" + getHandName(userRank) + ")");
            System.out.println(cpuDialogue(personality, "foldMid") + " ...you got me!");
        } else if (cpuRank > userRank) {
            System.out.println("\n🤖 CPU wins! (" + getHandName(cpuRank) + ")");
            System.out.println("🤖 Better luck next time!");
        } else {
            System.out.println("\n🤝 It's a tie!");
            System.out.println("🤖 Hah, evenly matched!");
        }
    }
}
