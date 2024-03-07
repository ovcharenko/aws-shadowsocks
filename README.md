# README

## Shadowsocks Server Deployment

This is an automated deployment of a Shadowsocks server using an AWS CloudFormation template. The server utilizes the latest Ubuntu version and is enhanced with Session Manager and VPC.

## Deployment Parameters

The deployment requires several parameters defined in the CloudFormation template:

* `ServerPort` - ServerPort will be used by the server for Shadowsocks. The default value is set to 45785.
* `Method` - Encryption method used for shadowsocks. Possible values include plain, none, aes-128-gcm, aes-256-gcm, chacha20-ietf-poly1305, 2022-blake3-aes-128-gcm, 2022-blake3-aes-256-gcm, 2022-blake3-chacha20-poly1305. Default is set to `2022-blake3-chacha20-poly1305`
* `Password` - This parameter used to define the SSM parameter path storing a key for the Shadowsocks. Default is `/shadowsocks/password`.
* `VpcCidrBlock` - CIDR block for the VPC. Default is set to 10.0.0.0/16.
* `UbuntuLatestAmiId` - This parameter is used to define the instance AMI to be deployed. The default value is set `/aws/service/canonical/ubuntu/server/22.04/stable/current/arm64/hvm/ebs-gp2/ami-id`.
* `InstanceTypeParam` - EC2 Instance Type. Possible values include t4g.nano, t4g.micro, t4g.small. Default is set to `t4g.small`.

## Deployment Steps

1. Navigate to AWS CloudFormation in AWS Console.
2. Choose "Create Stack".
3. In "Specify Template", choose "Upload a template file".
4. Click "Choose File" and select your cloudformation file.
5. Click "Next".
6. Specify the necessary details and parameters in "Specify Stack Details" and then click "Next".
7. Configure additional options if any and then click "Next".
8. Review your settings and acknowledge that AWS CloudFormation might create IAM resources, then click "Create Stack".
9. After the stack creation is complete, the IP address, Shadowsocks server port, and encryption method are provided in the "Outputs" tab of the CloudFormation stack.

## Client Configuration
The actions outlined are one-time procedures. The server, following restarts or recoveries, will continue to utilize the initially generated key.

1. Retrieve `/home/shadowsocks/ss.json` from the server using Connect to instance > Session manager session.
2. Modify the `server` variable with the server IP obtained from the CloudFormation stack output.
3. Insert `local_address` and `local_port` keys to the JSON. The final [configuration](https://shadowsocks.org/doc/configs.html) will appear as follows:
   ```json
    {
      "server": "...",
      "server_port": ...,
      "password": "...",
      "timeout": 300,
      "method": "2022-blake3-chacha20-poly1305",
      "local_address": "127.0.0.1",
      "local_port": 8008
    }
    ```

4. Install [client](https://github.com/shadowsocks/shadowsocks-rust)
5. Run `sslocal -c <configuration json file>` to start the client.
6. Adjust your system settings to use the above as SOCK5 proxy with the defined `local_address` and `local_port`.

## Security

The deployed stack includes a VPC with a Public Subnet and an Internet Gateway. The Shadowsocks server is deployed to this VPC and utilizes an EC2 InstanceProfile (with SSM and EC2 Policies) for AWS integration. The server is accessible via an EIP, and all communication to the server via the specified ServerPort is permitted through a SecurityGroup. Please ensure that the Password for the Shadowsocks server is stored securely.

## Support

This is an open-source template. Support for this deployment is provided on a "best effort" basis.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
