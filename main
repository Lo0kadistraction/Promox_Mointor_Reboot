const express = require('express');
const bodyParser = require('body-parser');
const { exec } = require('child_process');
const util = require('util');
const execAsync = util.promisify(exec);

const app = express();
const PORT = 3000; // Port where the server will listen

app.use(bodyParser.json());

app.post('/webhook', async (req, res) => {
    console.log('Webhook received:', req.body);

    if (req.body.heartbeat && req.body.heartbeat.status === 0) {
        const containerIdMap = {
            '1': '102', //sonarr
            '2': '103', //radarr
            '3': '104' //prowlarr
        };
        const monitorID = req.body.heartbeat.monitorID.toString();
        const containerId = containerIdMap[monitorID];

        if (!containerId) {
            console.error('No matching container ID found for monitorID:', monitorID);
            return res.status(400).send('No matching container ID found');
        }

        try {
            // Attempt to shutdown the container
            await execAsync(`pct shutdown ${containerId}`);
            console.log(`Container ${containerId} shutdown initiated.`);
        } catch (error) {
            console.error(`Error during shutdown: ${error}`);
            // Check if the error indicates that the container is not running
            if (error.stderr.includes("not running")) {
                console.log(`Container ${containerId} is not running. Attempting to start it.`);
            } else {
                // If the error is not because the container is stopped, return an error response
                return res.status(500).send(`Failed to shutdown container ${containerId}. Error: ${error}`);
            }
        }

        // Start the container
        try {
            await execAsync(`pct start ${containerId}`);
            console.log(`Container ${containerId} start initiated.`);
            res.send(`Container ${containerId} has been successfully started.`);
        } catch (startError) {
            console.error(`Error during start: ${startError}`);
            return res.status(500).send(`Failed to start container ${containerId}. Error: ${startError}`);
        }
    } else {
        res.send('No action taken, container is not down');
    }
});

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
