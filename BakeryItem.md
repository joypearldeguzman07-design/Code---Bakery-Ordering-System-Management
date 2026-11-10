package Proj;
import java.io.Serializable;

public abstract class BakeryItem implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private double price;
    private String category;

    public BakeryItem(String name, double price, String category) {
        setName(name);
        setPrice(price);
        this.category = category;
    }

    public String getName() { 
        return name; 
    }
    public double getPrice() { 
        return price; 
    }
    public String getCategory() { 
        return category; 
    }

    public void setName(String name) {
        if (name == null || name.trim().isEmpty())
            throw new IllegalArgumentException("Product name is required.");
        this.name = name.trim();
    }

    public void setPrice(double price) {
        if (!Double.isFinite(price) || price <= 0)
            throw new IllegalArgumentException("Price must be greater than zero.");
        this.price = price;
    }

    public abstract double calculatePrice(int quantity);

    @Override
    public String toString() {
        return name + " - PHP " + String.format("%.2f", price);
    }

    public static class Bread extends BakeryItem {
        private static final long serialVersionUID = 1L;
        public Bread(String name, double price) { 
            super(name, price, "Bread"); 
        }
        @Override public double calculatePrice(int quantity) {
            validateQuantity(quantity);
            return getPrice() * quantity;
        }
    }

    public static class Cake extends BakeryItem {
        private static final long serialVersionUID = 1L;
        public Cake(String name, double price) { 
            super(name, price, "Cake"); 
        }
        @Override public double calculatePrice(int quantity) {
            validateQuantity(quantity);
            return getPrice() * quantity;
        }
    }

    protected void validateQuantity(int quantity) {
        if (quantity < 1) throw new IllegalArgumentException("Quantity must be at least 1.");
    }
}
