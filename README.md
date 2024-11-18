resource "aws_vpc" "vpc1" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = "vpc1"
  }
}

# Define VPC 2
resource "aws_vpc" "vpc2" {
  cidr_block = "10.1.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = "vpc2"
  }
}

# Define the VPC Peering Connection
resource "aws_vpc_peering_connection" "peer" {
  vpc_id        = aws_vpc.vpc1.id
  peer_vpc_id    = aws_vpc.vpc2.id
  auto_accept    = false

  tags = {
    Name = "vpc1-to-vpc2"
  }
}

# Define VPC Peering Connection Accepter
resource "aws_vpc_peering_connection_accepter" "accept" {
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
  auto_accept               = true
  tags = {
    Name = "vpc2-to-vpc1"
  }
}

# Add route tables for VPC 1
resource "aws_route_table" "vpc1_route_table" {
  vpc_id = aws_vpc.vpc1.id
  route {
    cidr_block                = "10.1.0.0/16"
    vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
  }
  tags = {
    Name = "vpc1-route-table"
  }
}

resource "aws_route_table_association" "vpc1_association" {
  subnet_id      = aws_subnet.subnet1.id
  route_table_id = aws_route_table.vpc1_route_table.id
}

# Add route tables for VPC 2
resource "aws_route_table" "vpc2_route_table" {
  vpc_id = aws_vpc.vpc2.id
  route {
    cidr_block                = "10.0.0.0/16"
    vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
  }
  tags = {
    Name = "vpc2-route-table"
  }
}

resource "aws_route_table_association" "vpc2_association" {
  subnet_id      = aws_subnet.subnet2.id
  route_table_id = aws_route_table.vpc2_route_table.id
}

# Optionally, create subnets
resource "aws_subnet" "subnet1" {
  vpc_id            = aws_vpc.vpc1.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "ap-southeast-2a"
  tags = {
    Name = "subnet1"
  }
}

resource "aws_subnet" "subnet2" {
  vpc_id            = aws_vpc.vpc2.id
  cidr_block        = "10.1.1.0/24"
  availability_zone = "ap-southeast-2b"
  tags = {
    Name = "subnet2"
  }
}
