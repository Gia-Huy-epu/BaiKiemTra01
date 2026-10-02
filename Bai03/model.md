using System.ComponentModel;

namespace TechMartManager
{
    public class Category
    {
        public string CategoryId { get; set; }
        public string CategoryName { get; set; }
    }

    public class Product
    {
        public string ProductId { get; set; }
        public string ProductName { get; set; }
        public string CategoryId { get; set; }
        public decimal UnitPrice { get; set; }
        public int Quantity { get; set; }
        public string ImagePath { get; set; }
    }
}
