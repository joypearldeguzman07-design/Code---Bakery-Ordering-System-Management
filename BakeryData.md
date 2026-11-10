package Proj;
import java.io.*;
import java.util.ArrayList;

public final class BakeryData {
    private static final String PRODUCT_FILE = "bakery_products.dat";
    private static final String ORDER_FILE = "bakery_orders.dat";

    private BakeryData() { }

    @SuppressWarnings("unchecked")
    public static ArrayList<BakeryItem> loadProducts() {
        Object data = readObject(PRODUCT_FILE);
        if (data instanceof ArrayList<?>) {
            try { return (ArrayList<BakeryItem>) data; }
            catch (ClassCastException ignored) { }
        }
        ArrayList<BakeryItem> defaults = new ArrayList<>();
        defaults.add(new BakeryItem.Bread(" pandesal".trim(), 5.00));
        defaults.add(new BakeryItem.Bread("Spanish Bread", 12.00));
        defaults.add(new BakeryItem.Cake("Chocolate Cake Slice", 85.00));
        defaults.add(new BakeryItem.Cake("Cheesecake Slice", 95.00));
        return defaults;
    }

    @SuppressWarnings("unchecked")
    public static ArrayList<Order> loadOrders() {
        Object data = readObject(ORDER_FILE);
        if (data instanceof ArrayList<?>) {
            try { return (ArrayList<Order>) data; }
            catch (ClassCastException ignored) { }
        }
        return new ArrayList<>();
    }

    public static void saveProducts(ArrayList<BakeryItem> products) throws IOException {
        writeObject(PRODUCT_FILE, products);
    }

    public static void saveOrders(ArrayList<Order> orders) throws IOException {
        writeObject(ORDER_FILE, orders);
    }

    private static Object readObject(String file) {
        File source = new File(file);
        if (!source.exists()) return null;
        try (ObjectInputStream input = new ObjectInputStream(new FileInputStream(source))) {
            return input.readObject();
        } catch (IOException | ClassNotFoundException ex) {
            return null;
        }
    }

    private static void writeObject(String file, Object data) throws IOException {
        try (ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream(file))) {
            output.writeObject(data);
        }
    }
}
