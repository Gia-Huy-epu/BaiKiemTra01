using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.IO;
using System.Linq;
using System.Windows.Forms;

namespace TechMartManager
{
    public partial class Form1 : Form
    {
        private BindingList<Product> _products;
        private BindingSource _bindingSource;

        public Form1()
        {
            InitializeComponent();
            SetupDataGridView();
            LoadInitialData();
            SetupShortcuts();
        }

        private void SetupDataGridView()
        {
            dgvProducts.AutoGenerateColumns = false;
            dgvProducts.SelectionMode = DataGridViewSelectionMode.FullRowSelect;

            dgvProducts.Columns.Add(new DataGridViewTextBoxColumn { DataPropertyName = "ProductId", HeaderText = "Mã SP" });
            dgvProducts.Columns.Add(new DataGridViewTextBoxColumn { DataPropertyName = "ProductName", HeaderText = "Tên SP" });
            dgvProducts.Columns.Add(new DataGridViewTextBoxColumn { DataPropertyName = "CategoryId", HeaderText = "Danh Mục" });

            var priceCol = new DataGridViewTextBoxColumn { DataPropertyName = "UnitPrice", HeaderText = "Đơn Giá" };
            priceCol.DefaultCellStyle.Format = "N0";
            dgvProducts.Columns.Add(priceCol);

            dgvProducts.Columns.Add(new DataGridViewTextBoxColumn { DataPropertyName = "Quantity", HeaderText = "Số Lượng" });
        }

        private void LoadInitialData()
        {
            var categories = new List<Category>
            {
                new Category { CategoryId = "C01", CategoryName = "Điện thoại" },
                new Category { CategoryId = "C02", CategoryName = "Laptop" },
                new Category { CategoryId = "C03", CategoryName = "Phụ kiện" }
            };

            cboCategory.DataSource = categories;
            cboCategory.DisplayMember = "CategoryName";
            cboCategory.ValueMember = "CategoryId";

            _products = new BindingList<Product>();
            _bindingSource = new BindingSource { DataSource = _products };
            dgvProducts.DataSource = _bindingSource;

            UpdateStatus();
        }

        private void SetupShortcuts()
        {
            ToolStripMenuItem menuFile = new ToolStripMenuItem("File");
            ToolStripMenuItem menuExport = new ToolStripMenuItem("Export CSV", null, btnExportCSV_Click);
            menuExport.ShortcutKeys = Keys.Control | Keys.E;
            ToolStripMenuItem menuExit = new ToolStripMenuItem("Exit", null, btnExit_Click);
            menuExit.ShortcutKeys = Keys.Control | Keys.X;

            menuFile.DropDownItems.Add(menuExport);
            menuFile.DropDownItems.Add(menuExit);
            menuStrip1.Items.Add(menuFile);
        }

        private void UpdateStatus()
        {
            lblStatus.Text = $"Tổng số sản phẩm: {_products.Count}";
        }

        private bool ValidateInput()
        {
            errorProvider.Clear();
            bool isValid = true;

            if (string.IsNullOrWhiteSpace(txtProductName.Text))
            {
                errorProvider.SetError(txtProductName, "Tên sản phẩm không được để trống.");
                isValid = false;
            }

            if (!decimal.TryParse(txtUnitPrice.Text, out decimal price) || price <= 0)
            {
                errorProvider.SetError(txtUnitPrice, "Đơn giá phải lớn hơn 0.");
                isValid = false;
            }

            if (!int.TryParse(txtQuantity.Text, out int qty) || qty < 0)
            {
                errorProvider.SetError(txtQuantity, "Số lượng phải lớn hơn hoặc bằng 0.");
                isValid = false;
            }

            return isValid;
        }

        private void btnAdd_Click(object sender, EventArgs e)
        {
            if (!ValidateInput()) return;

            var newProduct = new Product
            {
                ProductId = txtProductId.Text,
                ProductName = txtProductName.Text,
                CategoryId = cboCategory.SelectedValue.ToString(),
                UnitPrice = decimal.Parse(txtUnitPrice.Text),
                Quantity = int.Parse(txtQuantity.Text),
                ImagePath = picAvatar.ImageLocation
            };

            _products.Add(newProduct);
            UpdateStatus();
            ClearInputs();
        }

        private void btnUpdate_Click(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow == null || !ValidateInput()) return;

            var product = (Product)dgvProducts.CurrentRow.DataBoundItem;
            product.ProductId = txtProductId.Text;
            product.ProductName = txtProductName.Text;
            product.CategoryId = cboCategory.SelectedValue.ToString();
            product.UnitPrice = decimal.Parse(txtUnitPrice.Text);
            product.Quantity = int.Parse(txtQuantity.Text);
            product.ImagePath = picAvatar.ImageLocation;

            _bindingSource.ResetBindings(false);
        }

        private void btnDelete_Click(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow == null) return;

            DialogResult result = MessageBox.Show("Bạn có chắc chắn muốn xóa sản phẩm này?", "Xác nhận xóa", MessageBoxButtons.YesNo, MessageBoxIcon.Question);
            if (result == DialogResult.Yes)
            {
                var product = (Product)dgvProducts.CurrentRow.DataBoundItem;
                _products.Remove(product);
                UpdateStatus();
                ClearInputs();
            }
        }

        private void dgvProducts_SelectionChanged(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow != null && dgvProducts.CurrentRow.DataBoundItem is Product product)
            {
                txtProductId.Text = product.ProductId;
                txtProductName.Text = product.ProductName;
                cboCategory.SelectedValue = product.CategoryId;
                txtUnitPrice.Text = product.UnitPrice.ToString();
                txtQuantity.Text = product.Quantity.ToString();

                if (!string.IsNullOrEmpty(product.ImagePath) && File.Exists(product.ImagePath))
                {
                    picAvatar.ImageLocation = product.ImagePath;
                }
                else
                {
                    picAvatar.ImageLocation = null;
                }
            }
        }

        private void btnChooseImage_Click(object sender, EventArgs e)
        {
            using (OpenFileDialog ofd = new OpenFileDialog())
            {
                ofd.Filter = "Image Files (*.png;*.jpg;*.jpeg)|*.png;*.jpg;*.jpeg";
                if (ofd.ShowDialog() == DialogResult.OK)
                {
                    picAvatar.ImageLocation = ofd.FileName;
                }
            }
        }

        private void txtSearch_TextChanged(object sender, EventArgs e)
        {
            var keyword = txtSearch.Text.ToLower();
            var filtered = _products.Where(p => p.ProductName.ToLower().Contains(keyword)).ToList();
            _bindingSource.DataSource = new BindingList<Product>(filtered);
        }

        private void btnExportCSV_Click(object sender, EventArgs e)
        {
            using (SaveFileDialog sfd = new SaveFileDialog())
            {
                sfd.Filter = "CSV files (*.csv)|*.csv";
                if (sfd.ShowDialog() == DialogResult.OK)
                {
                    using (StreamWriter sw = new StreamWriter(sfd.FileName, false, System.Text.Encoding.UTF8))
                    {
                        sw.WriteLine("Mã SP,Tên SP,Danh Mục,Đơn Giá,Số Lượng");
                        foreach (var p in _products)
                        {
                            sw.WriteLine($"{p.ProductId},{p.ProductName},{p.CategoryId},{p.UnitPrice},{p.Quantity}");
                        }
                    }
                    MessageBox.Show("Xuất file CSV thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
                }
            }
        }

        private void btnExit_Click(object sender, EventArgs e)
        {
            Application.Exit();
        }

        private void ClearInputs()
        {
            txtProductId.Clear();
            txtProductName.Clear();
            txtUnitPrice.Clear();
            txtQuantity.Clear();
            picAvatar.ImageLocation = null;
            txtProductId.Focus();
        }

        private void txtProductId_TextChanged(object sender, EventArgs e)
        {

        }

        private void cboCategory_SelectedIndexChanged(object sender, EventArgs e)
        {

        }

        private void txtUnitPrice_TextChanged(object sender, EventArgs e)
        {

        }

        private void label4_Click(object sender, EventArgs e)
        {

        }

        private void label6_Click(object sender, EventArgs e)
        {

        }
    }
}
