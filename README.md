# rds
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS CloudFormation sample template to creates an Amazon MYSQL RDS database
  instance.'
Parameters:
  DBName:
    Default: Batch11
    Description: The MYSQL RDS database name
    Type: String
    MinLength: '1'
    MaxLength: '64'
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: must begin with a letter and contain only alphanumeric
      characters.
  DBUser:
    NoEcho: 'true'
    Description: The database admin account username
    Type: String
    MinLength: '1'
    MaxLength: '16'
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: must begin with a letter and contain only alphanumeric
      characters.
  DBPassword:
    NoEcho: 'true'
    Description: The database admin account password
    Type: String
    MinLength: '1'
    MaxLength: '41'
    AllowedPattern: '[a-zA-Z0-9]+'
    ConstraintDescription: must contain only alphanumeric characters.
  DBAllocatedStorage:
    Default: '5'
    Description: The size of the database (Gb)
    Type: Number
    MinValue: '5'
    MaxValue: '1024'
    ConstraintDescription: value must be between 5 and 1024 Gb.
  DBInstanceClass:
    Description: The database instance type
    Type: String
    Default: db.t2.micro
    AllowedValues: [db.t1.micro, db.t2.micro, db.t2.small, db.t2.medium,
      db.t2.large]
    ConstraintDescription: must select a valid database instance type.
  MultiAZ:
    Description: Multi-AZ master database
    Type: String
    Default: 'false'
    AllowedValues: ['true', 'false']
    ConstraintDescription: must be true or false.
  VpcId:
    Default: vpc-03c09d2538cafa97a
    Type: String  
    Description: VPC for creating the MYSQL RDS
Resources:
  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: MYSQL RDS SG
      GroupDescription: Creating SecurityGroup for this Ec2 Instance with port 22
      VpcId: !Ref VpcId
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 3306
        ToPort: 3306
        CidrIp: 0.0.0.0/0         

  DBSubnets:
    Type: AWS::RDS::DBSubnetGroup
    Properties: 
      DBSubnetGroupDescription: Defining the subnets for creating MYSQL RDS
      DBSubnetGroupName: MYSQL RDS Subnet
      SubnetIds: 
        - subnet-0d565caec776dc84b
        - subnet-06dd1f259fe362a7d
        - subnet-0566de704da7a4618
      
  MasterDB:
    Type: AWS::RDS::DBInstance
    Properties:
      DBName: !Ref DBName
      AllocatedStorage: !Ref DBAllocatedStorage
      DBInstanceClass: !Ref DBInstanceClass
      Engine: MySQL
      MasterUsername: !Ref DBUser
      MasterUserPassword: !Ref DBPassword
      MultiAZ: !Ref MultiAZ
      Tags:
      - Key: Name
        Value: Master Database
      VPCSecurityGroups: 
        - Ref: DBSecurityGroup
      DBSubnetGroupName: !Ref DBSubnets 
    # DeletionPolicy: Snapshot
Outputs:
  MasterJDBCConnectionString:
    Description: JDBC connection string for the master database
    Value: !Join ['', ['jdbc:mysql://', !GetAtt [MasterDB, Endpoint.Address], ':',
        !GetAtt [MasterDB, Endpoint.Port], /, !Ref 'DBName']]     
