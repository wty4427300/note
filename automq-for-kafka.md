# auto-kafka用到的云原生技术

1. Spot Instances(https://aws.amazon.com/ec2/spot/): Offload storage to EBS and S3, and make the broker stateless to
   leverage Spot Instances for up to 90% cost savings.
2. ASG(https://aws.amazon.com/autoscaling/): The lightweight Kubernetes provided by the IaaS layer is simple yet
   powerful for achieving auto-scaling.
3. EBS & Multi-Attach(https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html): EBS ensures data
   durability with 3 replicas, freeing AutoMQ from managing replication. Each broker relies on a single EBS volume for
   WAL, with Multi-Attach ensuring resilience against instance failures.
4. NVMe
   Reservations(https://aws.amazon.com/blogs/aws/new-nvme-reservations-for-amazon-elastic-block-store-io2-volumes/):
   Addressing shared storage challenges is simpler with NVMe Reservations, which offer a straightforward way to
   implement Fencing.
5. Regional EBS(https://cloud.google.com/compute/docs/disks/regional-persistent-disk): Contrary to common belief, EBS
   can handle AZ-level failures, thanks to Regional EBS—a feature also available in both Azure and GCP.
6. S3 Express One Zone(https://aws.amazon.com/s3/storage-classes/express-one-zone/): This addition to the object storage
   family, when combined with EBS, allows AutoMQ to provide Regional WAL, filling a gap left by AWS's lack of Regional
   EBS.
7. S3: Obviously, we all need it.

