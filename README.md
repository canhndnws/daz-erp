# DAZ-ERP

This project is structured as a root Moqui component containing and managing multiple specialized modules within the `components/` directory.

### Project Structure
- `component.xml`: Root component definition for the ERP Shell.
- `components/daz-common`: Shared entities and logic.
- `components/daz-hr`: Human Resources functionality.
- `components/daz-crm`: Customer Relationship Management functionality.

## Deployment

To deploy all modules at once, simply copy/symlink this entire `daz-erp` directory into the `runtime/component` directory of your Moqui project.

```bash
# Example for Windows
New-Item -ItemType SymbolicLink -Path "C:\path\to\moqui\runtime\component\daz-erp" -Target "E:\example\daz-erp"
```
