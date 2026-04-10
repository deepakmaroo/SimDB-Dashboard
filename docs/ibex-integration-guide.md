# SimDB Dashboard - IBEX IDS Integration Implementation Guide

## ✅ Implementation Complete

The IBEX IDS integration has been successfully implemented in your SimDB Dashboard. Here's what was added:

---

## 📁 New Files Created

### 1. **API Service** (`src/api/ibexIdsAPI.ts`)
- `IBEXIdsAPI` class for communicating with IBEX backend
- Configured via proxy at `/api/ibex` (development)
- Methods:
  - `parseUri()` - Parse IMAS URIs
  - `listNodes()` - List all nodes in an IDS
  - `getNodeChildren()` - Get child nodes
  - `getFieldValue()` - Fetch field values for primitive types
  - `getPlotData()` - Fetch time-series data for plotting

### 2. **URI Utilities** (`src/utils/uriHelper.ts`)
- `parseIMASUri()` - Parse IMAS URI format
- `formatNodePath()` - Convert paths to readable labels
- `formatUri()` - Format URIs for display
- `isTimeSeriesUri()` - Detect time-series nodes
- `buildListUri()` / `buildNodeUri()` - Construct URIs for API calls
- `validateIMASUri()` - Validate URI format

### 3. **Components**

#### `IDSExplorer.vue` (Main Component)
- Handles URI parameter from route
- Fetches and displays list of nodes
- Auto-selects first node
- Error handling and loading states
- Breadcrumb navigation

#### `IDSNodeTree.vue` (Node Browser)
- Hierarchical tree view of IDS nodes
- Grouping by parent path
- Search/filter functionality
- Click to select node
- Shows data type and shape

#### `IDSNodePlot.vue` (Data Visualization)
- Displays time-series plots using Plotly
- Shows metadata (label, units, description)
- Statistics panel (min, max, mean, point count)
- Export data to CSV and JSON
- Responsive design

### 4. **Router Update** (`src/router/index.ts`)
- Added `/ids-explorer` route
- Named route: `ids-explorer`
- Accepts `uri` query parameter

### 5. **DetailView.vue Updates** 
- Imported `formatUri` from utilities
- Imported `useRouter` from Vue Router
- Added `navigateToIDS()` function - normalizes URIs and opens in new tab
- Added `isIMASUri()` helper function
- Made only IMAS URIs (starting with `imas:`) clickable in Inputs/Outputs
- Non-IMAS URIs display as plain text
- Added styling for URI cells with hover effects

---

## 🔄 User Flow

```
1. User selects a simulation from list
   ↓
2. DetailView shows simulation metadata
   ↓
3. User sees Inputs/Outputs URIs as clickable links
   ↓
4. Click on URI
   ↓
5. Router navigates to IDSExplorer with URI parameter
   ↓
6. IDSExplorer:
   - Parses URI
   - Fetches list of nodes from IBEX
   - Displays tree view
   - Auto-selects first node
   ↓
7. IDSNodeTree shows all available nodes
   ↓
8. User clicks node
   ↓
9. If time-series:
   - Fetches plot data from IBEX
   - Shows plot with Plotly
   - Displays statistics
   - Export options (CSV/JSON)
   ↓
10. If multi-dimensional:
    - Shows metadata
    - Shows shape and dtype
    - Displays info message
```

---

## 🚀 Getting Started

### 1. Verify IBEX Server is Running

You can check if the IBEX server is accessible by testing the version endpoint:

```bash
curl http://localhost:6060/ids_info/version
```

Expected response: `200 OK` with version information

### 2. Update Development Proxy (if needed)

Edit `src/config.ts` to match your IBEX server:

```typescript
ibexBackend: {
  host: 'your-host',
  port: 6060,
  protocol: 'http'
}
```

### 3. Test the Integration

1. Start the dashboard: `npm run dev`
2. Navigate to a simulation in DetailView
3. Look for "Inputs" or "Outputs" section
4. Click on any blue URI (IMAS URIs starting with `imas:`)
5. You should see the IDS Explorer page in a new tab

### 3. Example URI Format

```
imas:hdf5?path=/home/ITER/marood/public/imasdb/JINTRAC_SIMULATIONS/40acc396a74a11efa76fd4f5ef75e918/imasdb/iter/3/53301/2#summary:0/global_quantities/ip/value
```

Breakdown:
- **Protocol**: `imas:hdf5`
- **Path**: `/home/.../imasdb`
- **IDS**: `summary`
- **Occurrence**: `0`
- **Node**: `global_quantities/ip/value`

---

## 📊 API Endpoints Required

Your IBEX server at `http://localhost:6060` needs these endpoints:

### GET `/ids_info/version`
```
Parameters: none
Returns: Version information about the IBEX server
Note: Can be used to verify the server is running and accessible
```

### GET `/ids_info/list`
```
Parameters: uri (IMAS URI without node path)
Returns: { "nodes": [ { "path", "name", "dtype", "shape", "description", "units" } ] }
```

### GET `/ids_info/node_info`
```
Parameters: uri (full IMAS URI with node path)
Returns: { children array with node structure }
```

### GET `/data/field_value`
```
Parameters: 
  - uri (full IMAS URI)
  - downsampled_size (optional, not currently implemented)
Returns: 
  - String value for primitive types
  - Array for numeric arrays
  - Status 464 if no data available
```

### GET `/data/plot_data`
```
Parameters: 
  - uri (full IMAS URI)
  - downsampled_size (optional, not currently implemented)
Returns: TimeSeriesData object with values and metadata
```

---

## 🎯 Features Implemented

✅ **Node Browsing**
- Hierarchical tree view with lazy loading
- Caching for structure children
- Expandable/collapsible nodes
- Click to select and load data

✅ **Data Display**
- Primitive field values (strings, numbers)
- Numeric arrays as interactive plots
- Field metadata (path, dtype, shape, units)
- Special handling for "no data" scenarios (HTTP 464)

✅ **Data Caching**
- Structure node children cached by path
- Field value responses cached to avoid redundant API calls
- Automatic cache invalidation on new selections

✅ **URI Handling**
- Auto-normalization of old URI formats
- Default backend type (hdf5) for incomplete URIs
- Only IMAS URIs are clickable (others display as plain text)
- Opens in new browser tab

✅ **Error Handling**
- User-friendly error messages
- Loading states for async operations
- Special handling for "no data available" (HTTP 464)
- Graceful fallback for malformed data

✅ **Navigation**
- Breadcrumbs showing database/shot/run/ids
- Dynamic breadcrumb visibility (hides unknown values)
- Clickable links in DetailView Inputs/Outputs
- Back navigation through browser

---

## 🔧 Configuration

### IBEX Server URL

The IBEX backend is configured in `src/config.ts`:

```typescript
ibexBackend: {
  host: 'localhost',
  port: 6060,
  protocol: 'http'
}
```

**Development Setup** (`vite.config.ts`):
- Proxy path: `/api/ibex`
- Target: `http://{host}:{port}` (from config)
- Request rewrite: Removes `/api/ibex` prefix

To change the IBEX server URL, update the `ibexBackend` configuration in `src/config.ts`.

---

## 📝 Component Props

### IDSNodeTree
```typescript
interface Props {
  nodes: IDSNode[]           // List of available nodes
  selected: IDSNode | null   // Currently selected node
}

interface Emits {
  select: [node: IDSNode]    // Emitted when user selects a node
}
```

### IDSNodePlot
```typescript
interface Props {
  plotData: TimeSeriesData   // Data to plot
  node: IDSNode              // Node metadata
}
```

---

## 📈 Performance Considerations

1. **Large Datasets**: Downsampling parameter is optional and not currently implemented
2. **Caching**: Automatic caching of:
   - Structure node children (by node.path)
   - Field values and plot data (by full URI)
3. **Lazy Loading**: Child nodes loaded only when expanded
4. **Background Normalization**: URI normalization happens on demand without affecting display

---

## 🔐 Security Notes

- URIs contain full filesystem paths - don't expose in logs
- IBEX server should have authentication in production
- Consider rate limiting for API calls
- Validate URI format before making API calls

---

## 🐛 Troubleshooting

### Issue: "No URI provided"
**Solution**: The IDSExplorer component didn't receive a URI parameter
- Ensure you're clicking on blue IMAS URIs from DetailView
- Non-IMAS URIs (plain text) are not clickable
- Check that router is properly configured

### Issue: CORS Error
**Solution**: IBEX server needs CORS headers configured

**IBEX Backend (Python/Flask)**:
```python
from flask_cors import CORS
CORS(app, resources={
    r"/ids_info/*": {"origins": "*"},
    r"/data/*": {"origins": "*"}
})
```

### Issue: "Failed to load nodes"
**Solution**: IBEX endpoint not responding

1. Check IBEX backend config in `src/config.ts`
2. Verify the IBEX server is accessible at the configured host/port
3. Check browser console for detailed error
4. Verify URI format is valid

### Issue: "No data available" message
**Solution**: This is normal behavior for some nodes
- The backend returns HTTP 464 status to indicate no data
- The message from the server is displayed to the user
- This is not an error - it's informational

### Issue: Plot not showing
**Solution**: Node might be a structure type or unsupported format

- Structure types (STRUCT) show as expandable nodes
- Multi-dimensional arrays display as scalar values or text
- Check the node dtype in the UI
- Use browser console to see the actual response

---

