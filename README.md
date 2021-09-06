*
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
