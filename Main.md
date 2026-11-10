# Code---Bakery-Ordering-System-Management
De Guzman, Joypearl Marie C. &amp; Delimios, Angelica Mae M.

package Proj;
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.awt.print.PrinterException;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;


public class Main extends JFrame {
    private final Color cream = new Color(255, 248, 239);
    private final Color palePink = new Color(248, 220, 226);
    private final Color rose = new Color(183, 65, 103);
    private final Color brown = new Color(105, 70, 49);
    private final Color white = new Color(255, 253, 249);

    private final ArrayList<BakeryItem> products = BakeryData.loadProducts();
    private final ArrayList<Order> orders = BakeryData.loadOrders();
    private final ArrayList<BakeryItem> cart = new ArrayList<>();
    private final ArrayList<Integer> quantities = new ArrayList<>();

    private String currentUser = "";
    private String currentRole = "";
    private int nextOrderNumber = 1;

    private final CardLayout rootLayout = new CardLayout();
    private final JPanel root = new JPanel(rootLayout);
    private final CardLayout pageLayout = new CardLayout();
    private final JPanel pages = new JPanel(pageLayout);

    private JTextField usernameField;
    private JPasswordField passwordField;
    private JLabel roleLabel, productCount, orderCount, salesCount;
    private JTextField productNameField, productPriceField, customerField;
    private JComboBox<String> categoryBox;
    private JComboBox<BakeryItem> productBox;
    private JSpinner quantitySpinner;
    private JTable productTable, cartTable, orderTable;
    private DefaultTableModel productModel, cartModel, orderModel;
    private JPanel productManagerControls, recordManagerControls;
    private JLabel cartTotalLabel;

    public Main() {
        setTitle("Sweet Crumbs Bakery | Point of Sale");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(1120, 720);
        setMinimumSize(new Dimension(900, 620));
        setLocationRelativeTo(null);
        getContentPane().setBackground(cream);

        for (Order order : orders) {
            try {
                nextOrderNumber = Math.max(nextOrderNumber,
                    Integer.parseInt(order.getOrderId().replace("ORD", "")) + 1);
            } catch (NumberFormatException ignored) { }
        }

        root.add(createLoginPage(), "Login");
        root.add(createApplicationPage(), "App");
        setContentPane(root);
        rootLayout.show(root, "Login");
        persistProductsQuietly();
    }

    private DefaultTableModel tableModel(String... columns) {
        return new DefaultTableModel(columns, 0) {
            @Override public boolean isCellEditable(int row, int column) { return false; }
        };
    }

    private JPanel createLoginPage() {
        JPanel background = new JPanel(new GridBagLayout());
        background.setBackground(cream);

        // Main login card, centered in the window
        JPanel card = new JPanel(new GridBagLayout());
        card.setPreferredSize(new Dimension(560, 370));
        card.setBackground(white);
        card.setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(palePink, 2),
            BorderFactory.createEmptyBorder(24, 28, 24, 28)));

        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(7, 8, 7, 8);
        c.fill = GridBagConstraints.HORIZONTAL;

        JLabel title = new JLabel("SWEET CRUMBS", SwingConstants.CENTER);
        title.setFont(new Font("Serif", Font.BOLD, 32));
        title.setForeground(rose);

        JLabel subtitle = new JLabel("BAKERY POINT OF SALE", SwingConstants.CENTER);
        subtitle.setFont(new Font("Serif", Font.PLAIN, 15));
        subtitle.setForeground(brown);

        usernameField = new JTextField();
        passwordField = new JPasswordField();
        styleField(usernameField);
        styleField(passwordField);

        usernameField.setPreferredSize(new Dimension(340, 42));
        passwordField.setPreferredSize(new Dimension(340, 42));
        usernameField.setMaximumSize(new Dimension(340, 42));
        passwordField.setMaximumSize(new Dimension(340, 42));

        JLabel usernameLabel = label("Username");
        usernameLabel.setFont(new Font("Serif", Font.BOLD, 16));
        JLabel passwordLabel = label("Password");
        passwordLabel.setFont(new Font("Serif", Font.BOLD, 16));

        c.gridx = 0;
        c.gridy = 0;
        c.gridwidth = 2;
        c.weightx = 1;
        c.insets = new Insets(2, 8, 2, 8);
        card.add(title, c);

        c.gridy = 1;
        c.insets = new Insets(2, 8, 24, 8);
        card.add(subtitle, c);

        // Labels stay on the left side of their respective text boxes.
        c.gridwidth = 1;
        c.gridy = 2;
        c.gridx = 0;
        c.weightx = 0;
        c.fill = GridBagConstraints.NONE;
        c.anchor = GridBagConstraints.LINE_END;
        c.insets = new Insets(6, 8, 6, 14);
        card.add(usernameLabel, c);

        c.gridx = 1;
        c.weightx = 1;
        c.fill = GridBagConstraints.HORIZONTAL;
        c.anchor = GridBagConstraints.CENTER;
        c.insets = new Insets(6, 8, 6, 8);
        card.add(usernameField, c);

        c.gridy = 3;
        c.gridx = 0;
        c.weightx = 0;
        c.fill = GridBagConstraints.NONE;
        c.anchor = GridBagConstraints.LINE_END;
        c.insets = new Insets(6, 8, 6, 14);
        card.add(passwordLabel, c);

        c.gridx = 1;
        c.weightx = 1;
        c.fill = GridBagConstraints.HORIZONTAL;
        c.anchor = GridBagConstraints.CENTER;
        c.insets = new Insets(6, 8, 6, 8);
        card.add(passwordField, c);

        JButton login = button("LOG IN", this::login);
        login.setPreferredSize(new Dimension(172, 48));
        login.setFont(new Font("Serif", Font.BOLD, 15));
        c.gridx = 0;
        c.gridy = 4;
        c.gridwidth = 2;
        c.weightx = 0;
        c.fill = GridBagConstraints.NONE;
        c.anchor = GridBagConstraints.CENTER;
        c.insets = new Insets(22, 8, 2, 8);
        card.add(login, c);

        passwordField.addActionListener(e -> login());

        background.add(card);
        return background;
    }

    private JPanel createApplicationPage() {
        JPanel app = new JPanel(new BorderLayout(10, 10));
        app.setBackground(cream);
        app.setBorder(BorderFactory.createEmptyBorder(12, 16, 16, 16));

        JPanel header = new JPanel(new BorderLayout());
        header.setBackground(cream);
        JPanel branding = new JPanel();
        branding.setOpaque(false);
        branding.setLayout(new BoxLayout(branding, BoxLayout.Y_AXIS));
        JLabel brand = new JLabel("SWEET CRUMBS BAKERY");
        brand.setFont(new Font("Serif", Font.BOLD, 24));
        brand.setForeground(rose);
        roleLabel = new JLabel("Signed in");
        roleLabel.setForeground(brown);
        branding.add(brand);
        branding.add(roleLabel);
        header.add(branding, BorderLayout.WEST);

        JPanel navigation = new JPanel(new FlowLayout(FlowLayout.RIGHT, 6, 4));
        navigation.setOpaque(false);
        navigation.add(button("Dashboard", () -> showPage("Dashboard")));
        navigation.add(button("Products / Inventory", () -> showPage("Products")));
        navigation.add(button("New Order", () -> showPage("Orders")));
        navigation.add(button("Records", () -> showPage("Records")));
        navigation.add(button("Log Out", this::logout));
        header.add(navigation, BorderLayout.SOUTH);
        app.add(header, BorderLayout.NORTH);

        pages.setBackground(cream);
        pages.add(createDashboardPage(), "Dashboard");
        pages.add(createProductsPage(), "Products");
        pages.add(createOrdersPage(), "Orders");
        pages.add(createRecordsPage(), "Records");
        app.add(pages, BorderLayout.CENTER);
        return app;
    }

    private JPanel createDashboardPage() {
        JPanel page = pagePanel();
        JLabel welcome = new JLabel("Welcome to your bakery dashboard");
        welcome.setFont(new Font("Serif", Font.BOLD, 25));
        welcome.setForeground(brown);
        page.add(welcome, BorderLayout.NORTH);

        JPanel stats = new JPanel(new GridLayout(1, 3, 14, 14));
        stats.setOpaque(false);
        productCount = statValue();
        orderCount = statValue();
        salesCount = statValue();
        stats.add(statCard("PRODUCTS IN INVENTORY", productCount));
        stats.add(statCard("TOTAL ORDERS", orderCount));
        stats.add(statCard("TODAY'S SALES", salesCount));

        JPanel note = new JPanel(new GridBagLayout());
        note.setBackground(white);
        note.setBorder(BorderFactory.createLineBorder(palePink, 2));
        JLabel message = new JLabel("<html><center>Manage bakery products, prepare customer orders,<br>"
            + "print receipts, and review sales records.</center></html>");
        message.setForeground(brown);
        message.setFont(new Font("Serif", Font.PLAIN, 19));
        note.add(message);
        JPanel center = new JPanel(new BorderLayout(12, 20));
        center.setOpaque(false);
        center.add(stats, BorderLayout.NORTH);
        center.add(note, BorderLayout.CENTER);
        page.add(center, BorderLayout.CENTER);
        return page;
    }

    private JPanel createProductsPage() {
        JPanel page = pagePanel();
 
        productModel = tableModel("Product Name", "Category", "Price (PHP)");
        productTable = new JTable(productModel);
        styleTable(productTable);

        productNameField = new JTextField(12);
        productPriceField = new JTextField(10);
        categoryBox = new JComboBox<>(new String[]{"Bread", "Cake"});
        JPanel form = new JPanel(new GridBagLayout());
        form.setBackground(white);
        form.setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(palePink, 2),
            BorderFactory.createEmptyBorder(12, 12, 12, 12)));
        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(5, 5, 5, 5);
        c.fill = GridBagConstraints.HORIZONTAL;
        addFormRow(form, c, 0, "Product name", productNameField);
        addFormRow(form, c, 1, "Price (PHP)", productPriceField);
        addFormRow(form, c, 2, "Category", categoryBox);

        productManagerControls = new JPanel(new GridLayout(0, 1, 6, 6));
        productManagerControls.setOpaque(false);
        productManagerControls.add(button("Add Product", this::addProduct));
        productManagerControls.add(button("Update Selected", this::updateProduct));
        productManagerControls.add(button("Delete Selected", this::deleteProduct));
        productManagerControls.add(button("Clear Fields", this::clearProductFields));
        c.gridx = 0; c.gridy = 3; c.gridwidth = 2;
        form.add(productManagerControls, c);

        productTable.getSelectionModel().addListSelectionListener(e -> {
            if (!e.getValueIsAdjusting() && productTable.getSelectedRow() >= 0) {
                int row = productTable.convertRowIndexToModel(productTable.getSelectedRow());
                productNameField.setText(String.valueOf(productModel.getValueAt(row, 0)));
                categoryBox.setSelectedItem(String.valueOf(productModel.getValueAt(row, 1)));
                productPriceField.setText(String.valueOf(productModel.getValueAt(row, 2)));
            }
        });

        JPanel list = borderedPanel("Inventory List");
        list.add(new JScrollPane(productTable), BorderLayout.CENTER);
        JPanel body = new JPanel(new BorderLayout(12, 12));
        body.setOpaque(false);
        body.add(form, BorderLayout.WEST);
        body.add(list, BorderLayout.CENTER);
        page.add(body, BorderLayout.CENTER);
        return page;
    }

    private JPanel createOrdersPage() {
        JPanel page = pagePanel();
        page.add(sectionTitle("Customer Order"), BorderLayout.NORTH);
        customerField = new JTextField(14);
        productBox = new JComboBox<>();
        quantitySpinner = new JSpinner(new SpinnerNumberModel(1, 1, 999, 1));

        JPanel form = new JPanel(new GridBagLayout());
        form.setBackground(white);
        form.setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(palePink, 2),
            BorderFactory.createEmptyBorder(12, 12, 12, 12)));
        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(5, 5, 5, 5);
        c.fill = GridBagConstraints.HORIZONTAL;
        addFormRow(form, c, 0, "Customer name", customerField);
        addFormRow(form, c, 1, "Product", productBox);
        addFormRow(form, c, 2, "Quantity", quantitySpinner);

        JPanel actions = new JPanel(new GridLayout(0, 1, 6, 6));
        actions.setOpaque(false);
        actions.add(button("Add Item", this::addToCart));
        actions.add(button("Remove Selected Item", this::removeFromCart));
        actions.add(button("Calculate Total", this::calculateTotal));
        actions.add(button("Save Order + Receipt", this::saveOrder));
        actions.add(button("Clear Order", this::clearCart));
        c.gridx = 0; c.gridy = 3; c.gridwidth = 2;
        form.add(actions, c);

        cartModel = tableModel("Product", "Quantity", "Unit Price", "Subtotal");
        cartTable = new JTable(cartModel);
        styleTable(cartTable);
        JPanel cartPanel = borderedPanel("Current Order");
        cartPanel.add(new JScrollPane(cartTable), BorderLayout.CENTER);
        cartTotalLabel = new JLabel("TOTAL: PHP 0.00");
        cartTotalLabel.setFont(new Font("Serif", Font.BOLD, 20));
        cartTotalLabel.setForeground(rose);
        cartTotalLabel.setBorder(BorderFactory.createEmptyBorder(10, 8, 8, 8));
        cartPanel.add(cartTotalLabel, BorderLayout.SOUTH);

        JPanel body = new JPanel(new BorderLayout(12, 12));
        body.setOpaque(false);
        body.add(form, BorderLayout.WEST);
        body.add(cartPanel, BorderLayout.CENTER);
        page.add(body, BorderLayout.CENTER);
        return page;
    }

    private JPanel createRecordsPage() {
        JPanel page = pagePanel();
        page.add(sectionTitle("Sales Records"), BorderLayout.NORTH);
        orderModel = tableModel("Order ID", "Customer", "Items", "Total (PHP)", "Date & Time");
        orderTable = new JTable(orderModel);
        styleTable(orderTable);
        JPanel records = borderedPanel("Saved Orders");
        records.add(new JScrollPane(orderTable), BorderLayout.CENTER);

        recordManagerControls = new JPanel(new FlowLayout(FlowLayout.CENTER, 8, 8));
        recordManagerControls.setOpaque(false);
        recordManagerControls.add(button("View / Print Receipt", this::viewReceipt));
        recordManagerControls.add(button("Delete Selected Record", this::deleteOrder));
        recordManagerControls.add(button("Refresh Records", this::refreshAll));
        JPanel body = new JPanel(new BorderLayout(8, 8));
        body.setOpaque(false);
        body.add(records, BorderLayout.CENTER);
        body.add(recordManagerControls, BorderLayout.SOUTH);
        page.add(body, BorderLayout.CENTER);
        return page;
    }

    private void login() {
        String username = usernameField.getText().trim();
        String password = new String(passwordField.getPassword());
        if (username.equals("manager") && password.equals("manager123")) {
            currentUser = username;
            currentRole = "Manager";
        } else if (username.equals("cashier") && password.equals("cashier123")) {
            currentUser = username;
            currentRole = "Cashier";
        } else {
            message("Incorrect username or password.", JOptionPane.ERROR_MESSAGE);
            passwordField.setText("");
            return;
        }

        roleLabel.setText("Signed in as: " + currentUser + " (" + currentRole + ")");
        boolean manager = isManager();
        productManagerControls.setVisible(manager);
        recordManagerControls.getComponent(1).setVisible(manager);
        recordManagerControls.getComponent(0).setVisible(true);
        refreshAll();
        passwordField.setText("");
        rootLayout.show(root, "App");
        showPage("Dashboard");
    }

    private void logout() {
        int answer = JOptionPane.showConfirmDialog(this, "Are you sure you want to log out?",
            "Confirm Logout", JOptionPane.YES_NO_OPTION);
        if (answer != JOptionPane.YES_OPTION) return;
        currentUser = "";
        currentRole = "";
        clearCart();
        usernameField.setText("");
        passwordField.setText("");
        rootLayout.show(root, "Login");
    }

    private boolean isManager() { return "Manager".equals(currentRole); }

    private boolean managerOnly(String action) {
        if (!isManager()) {
            message("Access denied. Only the manager can " + action + ".", JOptionPane.ERROR_MESSAGE);
            return false;
        }
        return true;
    }

    private void addProduct() {
        if (!managerOnly("add products")) 
            return;
        
        try {
            String name = productNameField.getText().trim();
            double price = Double.parseDouble(productPriceField.getText().trim());
            String category = String.valueOf(categoryBox.getSelectedItem());
            if (name.isEmpty()) throw new IllegalArgumentException("Product name is required.");
            for (BakeryItem item : products)
                if (item.getName().equalsIgnoreCase(name))
                    throw new IllegalArgumentException("That product already exists.");
            products.add(makeItem(name, price, category));
            saveData();
            clearProductFields();
            refreshAll();
            message("Product added successfully.", JOptionPane.INFORMATION_MESSAGE);
        } catch (NumberFormatException ex) {
            message("Enter a valid numeric price.", JOptionPane.ERROR_MESSAGE);
        } catch (IllegalArgumentException ex) {
            message(ex.getMessage(), JOptionPane.ERROR_MESSAGE);
        }
    }

    private void updateProduct() {
        if (!managerOnly("update products")) 
            return;
        
        int selected = productTable.getSelectedRow();
        if (selected < 0) { message("Select a product first.", JOptionPane.WARNING_MESSAGE); return; }
        try {
            String name = productNameField.getText().trim();
            double price = Double.parseDouble(productPriceField.getText().trim());
            String category = String.valueOf(categoryBox.getSelectedItem());
            if (name.isEmpty()) throw new IllegalArgumentException("Product name is required.");
            int index = productTable.convertRowIndexToModel(selected);
            for (int i = 0; i < products.size(); i++)
                if (i != index && products.get(i).getName().equalsIgnoreCase(name))
                    throw new IllegalArgumentException("Another product already uses that name.");
            products.set(index, makeItem(name, price, category));
            saveData();
            clearProductFields();
            refreshAll();
        } catch (NumberFormatException ex) {
            message("Enter a valid numeric price.", JOptionPane.ERROR_MESSAGE);
        } catch (IllegalArgumentException ex) {
            message(ex.getMessage(), JOptionPane.ERROR_MESSAGE);
        }
    }

    private void deleteProduct() {
        if (!managerOnly("delete products")) return;
        int selected = productTable.getSelectedRow();
        if (selected < 0) { message("Select a product first.", JOptionPane.WARNING_MESSAGE); return; }
        int answer = JOptionPane.showConfirmDialog(this, "Delete the selected product?",
            "Confirm Delete", JOptionPane.YES_NO_OPTION);
        if (answer == JOptionPane.YES_OPTION) {
            products.remove(productTable.convertRowIndexToModel(selected));
            saveData();
            clearProductFields();
            refreshAll();
        }
    }

    private BakeryItem makeItem(String name, double price, String category) {
        if (price <= 0 || !Double.isFinite(price))
            throw new IllegalArgumentException("Price must be greater than zero.");
        return "Bread".equals(category) ? new BakeryItem.Bread(name, price)
                                         : new BakeryItem.Cake(name, price);
    }

    private void addToCart() {
        BakeryItem item = (BakeryItem) productBox.getSelectedItem();
        if (item == null) { message("There are no products in inventory.", JOptionPane.WARNING_MESSAGE); return; }
        int qty = (Integer) quantitySpinner.getValue();
        for (int i = 0; i < cart.size(); i++) {
            if (cart.get(i).getName().equalsIgnoreCase(item.getName())) {
                quantities.set(i, quantities.get(i) + qty);
                refreshCart();
                return;
            }
        }
        cart.add(item);
        quantities.add(qty);
        refreshCart();
    }

    private void removeFromCart() {
        int selected = cartTable.getSelectedRow();
        if (selected < 0) { message("Select an item from the current order.", JOptionPane.WARNING_MESSAGE); return; }
        int index = cartTable.convertRowIndexToModel(selected);
        cart.remove(index);
        quantities.remove(index);
        refreshCart();
    }

    private double
  cartTotal() {
        double total = 0;
        for (int i = 0; i < cart.size(); i++) total += cart.get(i).calculatePrice(quantities.get(i));
        return total;
    }

    private void calculateTotal() {
        message("Order total: PHP " + String.format("%.2f", cartTotal()), JOptionPane.INFORMATION_MESSAGE);
    }

    private void saveOrder() {
        if (cart.isEmpty()) { message("Add at least one product first.", JOptionPane.WARNING_MESSAGE); return; }
        String customer = customerField.getText().trim();
        if (customer.isEmpty()) customer = "Walk-in Customer";
        Order order = new Order(String.format("ORD%03d", nextOrderNumber++), customer);
        try {
            for (int i = 0; i < cart.size(); i++) order.addItem(cart.get(i), quantities.get(i));
            orders.add(order);
            saveData();
            refreshAll();
            showReceipt(order);
            clearCart();
        } catch (IllegalArgumentException ex) {
            message(ex.getMessage(), JOptionPane.ERROR_MESSAGE);
        }
    }

    private String receiptText(Order order) {
        StringBuilder receipt = new StringBuilder();
        receipt.append("====================================\n")
            .append("          SWEET CRUMBS BAKERY\n")
            .append("              SALES RECEIPT\n")
            .append("====================================\n")
            .append("Order ID: ").append(order.getOrderId()).append('\n')
            .append("Customer: ").append(order.getCustomerName()).append('\n')
            .append("Date: ").append(order.getDateTime()).append('\n')
            .append("------------------------------------\n");
        List<BakeryItem> items = order.getItems();
        List<Integer> qtys = order.getQuantities();
        List<Double> prices = order.getUnitPrices();
        for (int i = 0; i < items.size(); i++) {
            receipt.append(items.get(i).getName()).append('\n')
                .append(qtys.get(i)).append(" x PHP ")
                .append(String.format("%.2f", prices.get(i)))
                .append(" = PHP ")
                .append(String.format("%.2f", prices.get(i) * qtys.get(i))).append('\n');
        }
        return receipt.append("------------------------------------\n")
            .append("TOTAL: PHP ").append(String.format("%.2f", order.getTotal())).append('\n')
            .append("Thank you for shopping with us!\n")
            .append("====================================\n").toString();
    }

    private void showReceipt(Order order) {
        JTextArea area = new JTextArea(receiptText(order), 20, 35);
        area.setEditable(false);
        area.setFont(new Font("Monospaced", Font.PLAIN, 13));
        area.setBackground(cream);
        area.setForeground(brown);
        area.setMargin(new Insets(12, 12, 12, 12));
        int choice = JOptionPane.showOptionDialog(this, new JScrollPane(area),
            "Receipt - " + order.getOrderId(), JOptionPane.DEFAULT_OPTION,
            JOptionPane.INFORMATION_MESSAGE, null, new String[]{"Print Receipt", "Close"}, "Close");
        if (choice == 0) {
            try { area.print(); }
            catch (PrinterException ex) { message("Could not print receipt: " + ex.getMessage(), JOptionPane.ERROR_MESSAGE); }
        }
    }

    private void viewReceipt() {
        Order order = selectedOrder();
        if (order != null) showReceipt(order);
    }

    private void deleteOrder() {
        if (!managerOnly("delete sales records")) return;
        Order order = selectedOrder();
        if (order == null) return;
        int answer = JOptionPane.showConfirmDialog(this,
            "Permanently delete order " + order.getOrderId() + "?",
            "Confirm Delete", JOptionPane.YES_NO_OPTION);
        if (answer == JOptionPane.YES_OPTION) {
            orders.remove(order);
            saveData();
            refreshAll();
        }
    }

    private Order selectedOrder() {
        int selected = orderTable.getSelectedRow();
        if (selected < 0) { message("Select an order record first.", JOptionPane.WARNING_MESSAGE); return null; }
        String id = String.valueOf(orderModel.getValueAt(orderTable.convertRowIndexToModel(selected), 0));
        for (Order order : orders) if (order.getOrderId().equals(id)) return order;
        return null;
    }

    private void clearCart() {
        cart.clear();
        quantities.clear();
        if (customerField != null) customerField.setText("");
        refreshCart();
    }

    private void clearProductFields() {
        productNameField.setText("");
        productPriceField.setText("");
        categoryBox.setSelectedIndex(0);
        productTable.clearSelection();
    }

    private void refreshCart() {
        if (cartModel == null) return;
        cartModel.setRowCount(0);
        for (int i = 0; i < cart.size(); i++) {
            BakeryItem item = cart.get(i);
            int qty = quantities.get(i);
            cartModel.addRow(new Object[]{item.getName(), qty,
                String.format("%.2f", item.getPrice()),
                String.format("%.2f", item.calculatePrice(qty))});
        }
        cartTotalLabel.setText("TOTAL: PHP " + String.format("%.2f", cartTotal()));
    }

    private void refreshAll() {
        if (productModel == null || orderModel == null) return;
        productModel.setRowCount(0);
        productBox.removeAllItems();
        for (BakeryItem item : products) {
            productModel.addRow(new Object[]{item.getName(), item.getCategory(), String.format("%.2f", item.getPrice())});
            productBox.addItem(item);
        }
        orderModel.setRowCount(0);
        double todaySales = 0;
        for (Order order : orders) {
            orderModel.addRow(new Object[]{order.getOrderId(), order.getCustomerName(),
                order.getItemSummary(), String.format("%.2f", order.getTotal()), order.getDateTime()});
            if (order.getDateTime().startsWith(LocalDate.now().toString())) todaySales += order.getTotal();
        }
        productCount.setText(String.valueOf(products.size()));
        orderCount.setText(String.valueOf(orders.size()));
        salesCount.setText("PHP " + String.format("%.2f", todaySales));
        refreshCart();
    }

    private void saveData() {
        try {
            BakeryData.saveProducts(products);
            BakeryData.saveOrders(orders);
        } catch (Exception ex) {
            message("Could not save records: " + ex.getMessage(), JOptionPane.ERROR_MESSAGE);
        }
    }

    private void persistProductsQuietly() {
        try { BakeryData.saveProducts(products); }
        catch (Exception ex) { /* displayed if a later save is attempted */ }
    }

    private void showPage(String name) {
        if (currentUser.isEmpty()) { rootLayout.show(root, "Login"); return; }
        pageLayout.show(pages, name);
        refreshAll();
    }

    private JPanel pagePanel() {
        JPanel p = new JPanel(new BorderLayout(12, 12));
        p.setBackground(cream);
        p.setBorder(BorderFactory.createEmptyBorder(10, 4, 4, 4));
        return p;
    }

    private JPanel statCard(String title, JLabel value) {
        JPanel card = new JPanel(new BorderLayout(8, 12));
        card.setBackground(palePink);
        card.setBorder(BorderFactory.createCompoundBorder(BorderFactory.createLineBorder(rose),
            BorderFactory.createEmptyBorder(18, 10, 18, 10)));
        JLabel heading = new JLabel(title, SwingConstants.CENTER);
        heading.setForeground(brown);
        heading.setFont(new Font("Serif", Font.BOLD, 13));
        value.setHorizontalAlignment(SwingConstants.CENTER);
        value.setForeground(rose);
        value.setFont(new Font("Serif", Font.BOLD, 27));
        card.add(heading, BorderLayout.NORTH);
        card.add(value, BorderLayout.CENTER);
        return card;
    }

    private JLabel statValue() { return new JLabel("0", SwingConstants.CENTER); }

    private JPanel borderedPanel(String title) {
        JPanel p = new JPanel(new BorderLayout(8, 8));
        p.setBackground(white);
        p.setBorder(BorderFactory.createTitledBorder(
            BorderFactory.createLineBorder(palePink, 2), title));
        return p;
    }

    private JLabel sectionTitle(String text) {
        JLabel label = new JLabel(text);
        label.setFont(new Font("Serif", Font.BOLD, 22));
        label.setForeground(brown);
        return label;
    }

    private JLabel label(String text) {
        JLabel label = new JLabel(text);
        label.setForeground(brown);
        label.setAlignmentX(Component.LEFT_ALIGNMENT);
        return label;
    }

    private void addFormRow(JPanel panel, GridBagConstraints c, int row, String text, Component field) {
        c.gridwidth = 1;
        c.gridx = 0; c.gridy = row; c.weightx = 0;
        panel.add(label(text), c);
        c.gridx = 1; c.weightx = 1;
        panel.add(field, c);
    }

    private JPanel statCard(String title, JLabel value, boolean ignored) {
        return statCard(title, value);
    }

    private JButton button(String text, Runnable action) {
        JButton b = new JButton(text);
        b.setFocusPainted(false);
        b.setBackground(rose);
        b.setForeground(Color.WHITE);
        b.setBorder(BorderFactory.createEmptyBorder(9, 12, 9, 12));
        b.addActionListener(e -> action.run()); // ActionListener lesson
        return b;
    }

    private void styleField(JTextField field) {
        field.setMaximumSize(new Dimension(Integer.MAX_VALUE, 34));
        field.setPreferredSize(new Dimension(250, 34));
        field.setBackground(white);
        field.setForeground(brown);
    }

    private void styleTable(JTable table) {
        table.setRowHeight(27);
        table.setSelectionBackground(palePink);
        table.setSelectionForeground(brown);
        table.setGridColor(palePink);
        table.getTableHeader().setBackground(palePink);
        table.getTableHeader().setForeground(brown);
        table.setFillsViewportHeight(true);
    }

    private void message(String text, int type) {
        JOptionPane.showMessageDialog(this, text, "Sweet Crumbs Bakery", type);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new Main().setVisible(true));
    }
}
