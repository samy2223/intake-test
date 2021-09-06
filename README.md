const express = require('express');
const app = express();
const appRouter = express.Router();
const path = require('path');
const bodyParser = require('body-parser');
const nocache = require('nocache');
const port = process.env.PORT || 8080;
const AWS = require("aws-sdk");
const docClient = new AWS.DynamoDB.DocumentClient();
const pkg = require('./package.json');
const APP_PREFIX = pkg.deployment.appPrefix;
const { DB_NAME, DB_ITEM_PROJECT_LIST, DB_PROJECT_LIST, INTAKE_FORM } = require('./config.js');



/*
 * AWS and Express config
 */
AWS.config.update({ region: "us-east-1" });
app.use(bodyParser.json());
app.use(nocache());

/*
 * Get project list
 */
appRouter.get("/", function(req, res) {

	let param = {
		TableName: DB_NAME,
		KeyConditionExpression: 'id = :id',
		ExpressionAttributeValues: {
			':id': `${DB_PROJECT_LIST}#${DB_ITEM_PROJECT_LIST}`
		}
	};

	docClient.query(param, (err, data) => {
		if (err) {
			res.send(`Unable to query. Error: ${JSON.stringify(err, null, 2)}`);
		} else {
			res.send( data.Count ? `${JSON.stringify(data.Items[data.Count - 1])}` : 'No projects found');
		}
	});
});

/*
 * Send form
 */
appRouter.post("/:projectId", function(req, res) {
	let param = {
		TableName: DB_NAME,
		Item: {
			'id' : `${INTAKE_FORM}#${req.params.projectId}`,
			'date' : `${req.body.reportDate}`,
			'data': req.body.data
		}
	};

	if (req.body.changedBy) {
		param.Item.changed = req.body.changedBy
	}

	docClient.put(param, (err, data) => {
		if (err) {
			res.send(`Unable to query. Error: ${JSON.stringify(err, null, 2)}`);
		} else {
			res.send(`${JSON.stringify(data)}`);
		}
	});
});

/*
 * Get project items by ID
 */
appRouter.get("/:projectId", function(req, res) {
	let param = {
		TableName: DB_NAME,
		KeyConditionExpression: 'id = :id',
		ExpressionAttributeValues: {
			':id': `${INTAKE_FORM}#${req.params.projectId}`
		}
	};

	docClient.query(param, (err, data) => {
		if (err) {
			res.send(`Unable to query. Error: ${JSON.stringify(err, null, 2)}`);
		} else {
			let projectData = {};
			if (data.Count) {
				let projectItem = data.Items[data.Count - 1].data;
				projectData = { item: projectItem, count: data.Count };

				if (data.Items[data.Count - 1].changed) {
					projectData.changedBy = data.Items[data.Count - 1].changed;
				}
			} else {
				projectData = { item: {}, count: 0 };
			}
			res.json(projectData);
		}
	});
});

/*
 * Remove the X-Powered-By 'express' header and set 'Content-Security-Policy'
 */
app.use(function (request, response, next) {
	response.removeHeader('X-Powered-By');
	response.header('Content-Security-Policy', 'default-src \'self\' \'unsafe-eval\' https://static.vgcontent.info *.vanguard.com:8080 *.vanguard.com:1443 *.vanguard.com');
	response.header('Content-Security-Policy', 'img-src \'self\' *.vanguard.com:8080 *.vanguard.com:1443 *.vanguard.com');
	response.header('X-XSS-Protection', '1; mode=block');
	response.header('X-FRAME-OPTIONS', 'SAMEORIGIN');
	response.cookie('ALW-Log-Level', process.env.LOG_MANAGER_LEVEL || 0);
	next();
});

/*
 * Static application libraries.
 */
app.use(`/project`, appRouter);
app.use('/', express.static(path.resolve(__dirname, 'ui'), {fallthrough: false}));
app.listen(port, () => console.log(`App listening on port ${port}!`));

module.exports = app;



