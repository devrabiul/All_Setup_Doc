To create files and folders using a custom npm script in a React.js project, you can use Node.js and its file system module (fs). First, you'll need to create a JavaScript file that defines your custom command. Let's name it createFiles.js. Here's how you can structure it:

```bash
import fs from 'fs';
import path from 'path';

// Define your folder and file structure here
const structure = {
    src: {
        assets: {
            images: {},
            styles: {
                'styles.css': ''
            }
        },
        components: {
            Header: {
                'Header.jsx': '',
                'Header.css': ''
            },
            Footer: {
                'Footer.jsx': '',
                'Footer.css': ''
            },
            Sidebar: {
                'Sidebar.jsx': '',
                'Sidebar.css': ''
            },
            // Add more components as needed
        },
        pages: {
            Home: {
                'Home.jsx': '',
                'Home.css': ''
            },
            About: {
                'About.jsx': '',
                'About.css': ''
            },
            Contact: {
                'Contact.jsx': '',
                'Contact.css': ''
            },
            // Add more pages as needed
        },
        services: {
            'api.js': ''
        },
        utils: {
            'helpers.js': ''
        },
        context: {
            'AuthContext.js': ''
        },
        hooks: {
            'useFetch.js': ''
        },
        constants: {
            'actionTypes.js': ''
        },
    }
};

// Function to create directories and files recursively
const createStructure = (basePath, structure) => {
    const logs = [];
    const create = (basePath, structure, logs) => {
        Object.entries(structure).forEach(([name, content]) => {
            const fullPath = path.join(basePath, name);
            if (typeof content === 'object') {
                if (!fs.existsSync(fullPath)) {
                    fs.mkdirSync(fullPath, {
                        recursive: true
                    });
                    logs.push(`Folder "${name}" created.`);
                    create(fullPath, content, logs);
                } else {
                    logs.push(`Folder "${name}" already exists. Skipping...`);
                    create(fullPath, content, logs);
                }
            } else {
                if (!fs.existsSync(fullPath)) {
                    fs.writeFileSync(fullPath, content);
                    logs.push(`File "${name}" created.`);
                } else {
                    logs.push(`File "${name}" already exists. Skipping...`);
                }
            }
        });
    };
    create(basePath, structure, logs);
    return logs;
};

// Start creating the structure from the root directory
const logs = createStructure('.', structure);

logs.forEach(log => console.log(log));
```

Next, you need to modify your package.json to include a custom npm script that runs this file. Add the following line to the scripts section:

```bash
"custom-command": "node createFiles.js"
```

Now, whenever you run npm run custom-command, it will execute the createFiles.js script and generate the folder and file structure defined in your structure object.
