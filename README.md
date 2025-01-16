# daily-expense-trackerr
import java.io.*;
import java.time.*;
import java.util.*;

// Class to represent individual expenses
class Expense implements Serializable {
    private double amount;
    private String category;
    private String description;
    private LocalDate date;

    public Expense(double amount, String category, String description) {
        this.amount = amount;
        this.category = category;
        this.description = description;
        this.date = LocalDate.now();
    }

    // Getters
    public double getAmount() { return amount; }
    public String getCategory() { return category; }
    public String getDescription() { return description; }
    public LocalDate getDate() { return date; }

    @Override
    public String toString() {
        return String.format("Date: %s, Amount: $%.2f, Category: %s, Description: %s",
                date, amount, category, description);
    }
}

// Class to manage all expense-related operations
class ExpenseManager {
    private List<Expense> expenses;
    private static final String FILE_PATH = "expenses.txt";

    public ExpenseManager() {
        expenses = new ArrayList<>();
        loadExpenses();
    }

    // Add new expense
    public void addExpense(double amount, String category, String description) {
        Expense expense = new Expense(amount, category, description);
        expenses.add(expense);
        saveExpenses();
    }

    // Get expenses for specific period
    public List<Expense> getExpensesForPeriod(LocalDate start, LocalDate end) {
        return expenses.stream()
                .filter(e -> !e.getDate().isBefore(start) && !e.getDate().isAfter(end))
                .toList();
    }

    // Calculate total expenses for a period
    public double calculateTotalForPeriod(LocalDate start, LocalDate end) {
        return getExpensesForPeriod(start, end).stream()
                .mapToDouble(Expense::getAmount)
                .sum();
    }

    // Get daily expenses
    public double getDailyExpenses() {
        LocalDate today = LocalDate.now();
        return calculateTotalForPeriod(today, today);
    }

    // Get weekly expenses
    public double getWeeklyExpenses() {
        LocalDate today = LocalDate.now();
        LocalDate weekStart = today.minusDays(6);
        return calculateTotalForPeriod(weekStart, today);
    }

    // Get monthly expenses
    public double getMonthlyExpenses() {
        LocalDate today = LocalDate.now();
        LocalDate monthStart = today.withDayOfMonth(1);
        return calculateTotalForPeriod(monthStart, today);
    }

    // Save expenses to file
    private void saveExpenses() {
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream(FILE_PATH))) {
            oos.writeObject(expenses);
        } catch (IOException e) {
            System.err.println("Error saving expenses: " + e.getMessage());
        }
    }

    // Load expenses from file
    @SuppressWarnings("unchecked")
    private void loadExpenses() {
        File file = new File(FILE_PATH);
        if (!file.exists()) {
            return;
        }

        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream(FILE_PATH))) {
            expenses = (List<Expense>) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            System.err.println("Error loading expenses: " + e.getMessage());
        }
    }
}

// Main application class
public class DailyExpenseTracker {
    private static final Scanner scanner = new Scanner(System.in);
    private static final ExpenseManager expenseManager = new ExpenseManager();

    public static void main(String[] args) {
        while (true) {
            displayMenu();
            int choice = getChoice();

            switch (choice) {
                case 1 -> addNewExpense();
                case 2 -> viewSummary();
                case 3 -> {
                    System.out.println("Exiting application...");
                    return;
                }
                default -> System.out.println("Invalid choice. Please try again.");
            }
        }
    }

    private static void displayMenu() {
        System.out.println("\n=== Daily Expense Tracker ===");
        System.out.println("1. Add New Expense");
        System.out.println("2. View Summary");
        System.out.println("3. Exit");
        System.out.print("Enter your choice: ");
    }

    private static int getChoice() {
        try {
            return Integer.parseInt(scanner.nextLine());
        } catch (NumberFormatException e) {
            return -1;
        }
    }

    private static void addNewExpense() {
        System.out.print("Enter amount: $");
        double amount = Double.parseDouble(scanner.nextLine());

        System.out.print("Enter category (food/travel/utilities/other): ");
        String category = scanner.nextLine();

        System.out.print("Enter description: ");
        String description = scanner.nextLine();

        expenseManager.addExpense(amount, category, description);
        System.out.println("Expense added successfully!");
    }

    private static void viewSummary() {
        System.out.println("\n=== Expense Summary ===");
        System.out.printf("Daily Total: $%.2f%n", expenseManager.getDailyExpenses());
        System.out.printf("Weekly Total: $%.2f%n", expenseManager.getWeeklyExpenses());
        System.out.printf("Monthly Total: $%.2f%n", expenseManager.getMonthlyExpenses());
    }
} 
