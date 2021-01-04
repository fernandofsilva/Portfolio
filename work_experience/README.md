# create boto3 client object for lambda with timeout
    config = Config(connect_timeout=6000, retries={'max_attempts': 0})
    lambda_client = boto3.client('lambda', config=config)

    # create boto3 iam object
    iam_client = boto3.client('iam')
    role = iam_client.get_role(RoleName='lambda_prototype')

    with open('MessagesSocialNetworks.zip', 'rb') as f:
        zipped_code = f.read()

    if 'create' in action:
        # Create lambda_client lambda function
        response = lambda_client.create_function(
            FunctionName='SocialNetworksMessages',
            Description="Function to get posts from different social networks",
            Runtime='python3.8',
            Handler='main.handler',
            Role=role['Role']['Arn'],
            Code=dict(ZipFile=zipped_code),
            Environment=dict(Variables=env_variables),
        )
        print(response)
    elif 'modify' in action:
        response = lambda_client.update_function_code(
            FunctionName='SocialNetworksMessages',
            ZipFile=zipped_code,
        )
        print(response)
    elif 'run' in action:
        response = lambda_client.invoke(
            FunctionName='SocialNetworksMessages',
            InvocationType='RequestResponse')
        print(response["Payload"].read())
    elif 'delete' in action:
        response = lambda_client.delete_function(
            FunctionName='SocialNetworksMessages')
        print(response)
    else:
        print("Do nothing!")
