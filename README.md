-- ================================
-- DATABASE
-- ================================
DROP DATABASE IF EXISTS ShopDB; 
CREATE DATABASE ShopDB;
USE ShopDB;

-- ================================
-- CATEGORY
-- ================================
CREATE TABLE Category (
    CategoryID INT AUTO_INCREMENT PRIMARY KEY,
    CategoryName VARCHAR(255) NOT NULL
);

-- ================================
-- SUPPLIER
-- ================================
CREATE TABLE Supplier (
    SupplierID INT AUTO_INCREMENT PRIMARY KEY,
    SupplierName VARCHAR(255) NOT NULL,
    ContactName VARCHAR(255),
    Address VARCHAR(255),
    City VARCHAR(255),
    PostalCode VARCHAR(50),
    Country VARCHAR(255),
    Phone VARCHAR(50)
);

-- ================================
-- PRODUCT
-- ================================
CREATE TABLE Product (
    ProductID INT AUTO_INCREMENT PRIMARY KEY,
    ProductName VARCHAR(255) NOT NULL,
    SupplierID INT,
    CategoryID INT,
    UnitPrice DECIMAL(10,2),
    UnitsInStock INT,
    UnitsOnOrder INT,
    ReorderLevel INT,
    Discontinued CHAR(1),
    FOREIGN KEY (SupplierID) REFERENCES Supplier(SupplierID) ON DELETE SET NULL,
    FOREIGN KEY (CategoryID) REFERENCES Category(CategoryID) ON DELETE SET NULL
);

-- ================================
-- CUSTOMER
-- ================================
CREATE TABLE Customer (
    CustomerID INT AUTO_INCREMENT PRIMARY KEY,
    CustomerName VARCHAR(255) NOT NULL,
    ContactName VARCHAR(255),
    Address VARCHAR(255),
    City VARCHAR(255),
    PostalCode VARCHAR(50),
    Country VARCHAR(255),
    Phone VARCHAR(50),
    Email VARCHAR(255)
);

-- ================================
-- EMPLOYEE
-- ================================
CREATE TABLE Employee (
    EmployeeID INT AUTO_INCREMENT PRIMARY KEY,
    LastName VARCHAR(255) NOT NULL,
    FirstName VARCHAR(255) NOT NULL,
    BirthDate DATE,
    Photo VARCHAR(255),
    Notes TEXT
);

-- ================================
-- SHIPPER
-- ================================
CREATE TABLE Shipper (
    ShipperID INT AUTO_INCREMENT PRIMARY KEY,
    ShipperName VARCHAR(255) NOT NULL,
    Phone VARCHAR(50)
);

-- ================================
-- ORDERS
-- ================================
CREATE TABLE Orders (
    OrderID INT AUTO_INCREMENT PRIMARY KEY,
    CustomerID INT,
    EmployeeID INT,
    OrderDate DATE,
    ShipperID INT,
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID) ON DELETE SET NULL,
    FOREIGN KEY (EmployeeID) REFERENCES Employee(EmployeeID) ON DELETE SET NULL,
    FOREIGN KEY (ShipperID) REFERENCES Shipper(ShipperID) ON DELETE SET NULL
);

-- ================================
-- ORDER DETAILS
-- ================================
CREATE TABLE OrderDetails (
    OrderDetailID INT AUTO_INCREMENT PRIMARY KEY,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    Discount DECIMAL(10,2),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID) ON DELETE CASCADE,
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID) ON DELETE SET NULL
);

-- ================================
-- INVENTORY
-- ================================
CREATE TABLE Inventory (
    InventoryID INT AUTO_INCREMENT PRIMARY KEY,
    ProductID INT,
    Quantity INT,
    Location VARCHAR(255),
    LastUpdate DATE,
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID) ON DELETE SET NULL
);

-- ================================
-- PAYMENT
-- ================================
CREATE TABLE Payment (
    PaymentID INT AUTO_INCREMENT PRIMARY KEY,
    OrderID INT,
    PaymentDate DATE,
    PaymentAmount DECIMAL(10,2),
    PaymentMethod VARCHAR(50),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID) ON DELETE CASCADE
);

-- ================================
-- RETURN TABLE (FIXED NAME)
-- ================================
CREATE TABLE ReturnTable (
    ReturnID INT AUTO_INCREMENT PRIMARY KEY,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    ReturnDate DATE,
    Reason VARCHAR(255),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID) ON DELETE CASCADE,
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID) ON DELETE SET NULL
);

-- ================================
-- SUPPLIER PRODUCT
-- ================================
CREATE TABLE SupplierProduct (
    SupplierProductID INT AUTO_INCREMENT PRIMARY KEY,
    SupplierID INT,
    ProductID INT,
    FOREIGN KEY (SupplierID) REFERENCES Supplier(SupplierID) ON DELETE CASCADE,
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID) ON DELETE CASCADE
);

-- ================================
-- DISCOUNT
-- ================================
CREATE TABLE Discount (
    DiscountID INT AUTO_INCREMENT PRIMARY KEY,
    DiscountName VARCHAR(255),
    DiscountPercentage DECIMAL(5,2),
    StartDate DATE,
    EndDate DATE
);

-- ================================
-- PRODUCT DISCOUNT
-- ================================
CREATE TABLE ProductDiscount (
    ProductDiscountID INT AUTO_INCREMENT PRIMARY KEY,
    ProductID INT,
    DiscountID INT,
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID) ON DELETE CASCADE,
    FOREIGN KEY (DiscountID) REFERENCES Discount(DiscountID) ON DELETE CASCADE
);

-- ================================
-- CUSTOMER FEEDBACK
-- ================================
CREATE TABLE CustomerFeedback (
    FeedbackID INT AUTO_INCREMENT PRIMARY KEY,
    CustomerID INT,
    ProductID INT,
    FeedbackDate DATE,
    Rating INT,
    Comments TEXT,
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID) ON DELETE CASCADE,
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID) ON DELETE CASCADE
);

-- ================================
-- TRIGGERS
-- ================================

DELIMITER //

CREATE TRIGGER Product_Update_Trigger
AFTER UPDATE ON Product
FOR EACH ROW
BEGIN
    UPDATE OrderDetails SET ProductID = NEW.ProductID WHERE ProductID = OLD.ProductID;
    UPDATE Inventory SET ProductID = NEW.ProductID WHERE ProductID = OLD.ProductID;
    UPDATE ReturnTable SET ProductID = NEW.ProductID WHERE ProductID = OLD.ProductID;
    UPDATE SupplierProduct SET ProductID = NEW.ProductID WHERE ProductID = OLD.ProductID;
    UPDATE ProductDiscount SET ProductID = NEW.ProductID WHERE ProductID = OLD.ProductID;
    UPDATE CustomerFeedback SET ProductID = NEW.ProductID WHERE ProductID = OLD.ProductID;
END //

CREATE TRIGGER Supplier_Update_Trigger
AFTER UPDATE ON Supplier
FOR EACH ROW
BEGIN
    UPDATE Product SET SupplierID = NEW.SupplierID WHERE SupplierID = OLD.SupplierID;
    UPDATE SupplierProduct SET SupplierID = NEW.SupplierID WHERE SupplierID = OLD.SupplierID;
END //

CREATE TRIGGER Category_Update_Trigger
AFTER UPDATE ON Category
FOR EACH ROW
BEGIN
    UPDATE Product SET CategoryID = NEW.CategoryID WHERE CategoryID = OLD.CategoryID;
END //

CREATE TRIGGER Customer_Update_Trigger
AFTER UPDATE ON Customer
FOR EACH ROW
BEGIN
    UPDATE Orders SET CustomerID = NEW.CustomerID WHERE CustomerID = OLD.CustomerID;
    UPDATE CustomerFeedback SET CustomerID = NEW.CustomerID WHERE CustomerID = OLD.CustomerID;
END //

CREATE TRIGGER Employee_Update_Trigger
AFTER UPDATE ON Employee
FOR EACH ROW
BEGIN
    UPDATE Orders SET EmployeeID = NEW.EmployeeID WHERE EmployeeID = OLD.EmployeeID;
END //

CREATE TRIGGER Shipper_Update_Trigger
AFTER UPDATE ON Shipper
FOR EACH ROW
BEGIN
    UPDATE Orders SET ShipperID = NEW.ShipperID WHERE ShipperID = OLD.ShipperID;
END //

DELIMITER ;

-- ================================
-- SHOW TABLES
-- ================================
SHOW TABLES;
