package Proj;
import java.io.Serializable;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Order implements Serializable {
    private static final long serialVersionUID = 1L;

    private final String orderId;
    private String customerName;
    private final String dateTime;
    private final ArrayList<BakeryItem> items = new ArrayList<>();
    private final ArrayList<Integer> quantities = new ArrayList<>();
    private final ArrayList<Double> unitPrices = new ArrayList<>();

    public Order(String orderId, String customerName) {
        if (orderId == null || orderId.trim().isEmpty())
            throw new IllegalArgumentException("Order ID is required.");
        this.orderId = orderId.trim();
        setCustomerName(customerName);
        dateTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    }

    // Overloaded constructor
    public Order(BakeryItem item, int quantity) {
        this("ORD000", "Walk-in Customer");
        addItem(item, quantity);
    }

    public String getOrderId() {
        return orderId;
    }
    public String getCustomerName() {
        return customerName;
    }
    public String getDateTime() { 
        return dateTime;
    }
    public List<BakeryItem> getItems() { 
        return Collections.unmodifiableList(items); 
    }
    public List<Integer> getQuantities() {
        return Collections.unmodifiableList(quantities);
    }
    public List<Double> getUnitPrices() {
        return Collections.unmodifiableList(unitPrices);
    }

    public void setCustomerName(String name) {
        if (name == null || name.trim().isEmpty())
            throw new IllegalArgumentException("Customer name is required.");
        customerName = name.trim();
    }

    public void addItem(BakeryItem item, int quantity) {
        if (item == null || quantity < 1)
            throw new IllegalArgumentException("Select a product and enter a valid quantity.");
        for (int i = 0; i < items.size(); i++) {
            if (items.get(i).getName().equalsIgnoreCase(item.getName())) {
                quantities.set(i, quantities.get(i) + quantity);
                return;
            }
        }
        items.add(item);
        quantities.add(quantity);
        unitPrices.add(item.getPrice());
    }

    public void removeItem(int index) {
        if (index < 0 || index >= items.size())
            throw new IllegalArgumentException("Select a valid cart item.");
        items.remove(index);
        quantities.remove(index);
        unitPrices.remove(index);
    }

    public double getTotal() {
        double total = 0;
        for (int i = 0; i < items.size(); i++) total += unitPrices.get(i) * quantities.get(i);
        return total;
    }

    public String getItemSummary() {
        StringBuilder text = new StringBuilder();
        for (int i = 0; i < items.size(); i++) {
            if (i > 0) text.append(", ");
            text.append(items.get(i).getName()).append(" x").append(quantities.get(i));
        }
        return text.toString();
    }
}
