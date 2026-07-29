async function loadInventory() {
  const response = await fetch('inventory.csv');
  const text = await response.text();
  const rows = text.trim().split('\n');
  const headers = rows[0].split(',');
  
  let inventory = rows.slice(1).map(row => {
    const values = row.split(',');
    return headers.reduce((obj, header, i) => {
      obj[header.trim()] = values[i].trim();
      return obj;
    }, {});
  });

  // Remove duplicates (keep first occurrence by item_name)
  const seen = new Set();
  inventory = inventory.filter(item => {
    if (seen.has(item.item_name)) return false;
    seen.add(item.item_name);
    return true;
  });

  return inventory;
}

function checkExpired(item) {
  return new Date(item.expiration_date) < new Date();
}

function needsReorder(item) {
  return parseInt(item.quantity) <= parseInt(item.reorder_point);
}

function displayInventory(items) {
  const container = document.getElementById('inventory-container');
  container.innerHTML = items.map(item => {
    const expired = checkExpired(item);
    const lowStock = needsReorder(item);
    return `
      <div class="inventory-card ${expired ? 'expired' : ''} ${lowStock ? 'low-stock' : ''}">
        <h3>${item.item_name} ${expired ? '⚠️ EXPIRED' : ''} ${lowStock ? '🔄 REORDER' : ''}</h3>
        <p>Qty: ${item.quantity} | Reorder at: ${item.reorder_point}</p>
        <p>Supplier: ${item.supplier}</p>
        <p>Expires: ${item.expiration_date}</p>
      </div>
    `;
  }).join('');
}

loadInventory().then(displayInventory);
.expired { border: 2px solid red; }
.low-stock { border: 2px solid orange; }
