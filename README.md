Get project list
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
