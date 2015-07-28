VPN
===

### Create the gateway and a tunnel

    PROJECT=my-project
    VPNGATEWAY=vpn1
    NETWORK=default
    REGION=us-central1
    STATICIP=vpn-static-ip
    TUNNEL=tunnel1

    CUST_GW_EXT_IP=
    SHAREDSECRET=
    CIDR_DEST_RANGE=

    ## Create network if you need to.
    gcloud compute --project ${PROJECT} networks create \
        --range 10.120.0.0/16 ${NETWORK}

    ## Create the VPN Gateway
    gcloud compute --project ${PROJECT} target-vpn-gateways create \
        --region ${REGION} ${VPNGATEWAY}

    # Create the VPN static IP address
    gcloud compute --project ${PROJECT} addresses create \
        --region ${REGION} ${STATICIP}

    # Create forwarding rules that pass all VPN traffic to your VPN gateway
    gcloud compute --project ${PROJECT} forwarding-rules create \
        --region ${REGION} \
        --ip-protocol ESP \
        --address VPN_STATIC_IP_ADDRESS \
        --target-vpn-gateway ${VPNGATEWAY} fr-esp

    gcloud compute --project ${PROJECT} forwarding-rules create \
        --region ${REGION} \
        --ip-protocol UDP \
        --port 500 \
        --address VPN_STATIC_IP_ADDRESS \
        --target-vpn-gateway ${VPNGATEWAY} fr-udp500

    gcloud compute --project ${PROJECT} forwarding-rules create \
        --region ${REGION} \
        --ip-protocol UDP \
        --port 4500 \
        --address VPN_STATIC_IP_ADDRESS \
        --target-vpn-gateway ${VPNGATEWAY} fr-udp450

    # Create the VPN tunnel, this is done for each remote VPN tunnel
    gcloud compute --project ${PROJECT} vpn-tunnels create \
        --region ${REGION} \
        --peer-address ${CUST_GW_EXT_IP} \
        --ike-version 2 \
        --shared-secret ${SHAREDSECRET} \
        --target-vpn-gateway ${VPNGATEWAY} ${TUNNEL}

    # Create routes to the remote subnet behind the tunnel
    gcloud compute --project ${PROJECT} routes create \
        --network ${NETWORK} \
        --next-hop-vpn-tunnel ${TUNNEL} \
        --next-hop-vpn-tunnel-region ${REGION} \
        --destination-range ${CIDR_DEST_RANGE} ${TUNNEL}-route1

### Configure firewall rules

We now need to configure firewall rules to allow traffic to pass to/from the remote network that sits behind the VPN tunnel.

    gcloud  compute --project ${PROJECT} firewall-rules create ${TUNNEL}-vpnrule1 \
        --network ${NETWORK} \
        --allow tcp udp icmp \
        --source-ranges ${CIDR_DEST_RANGE}

### Check the status of your tunnel

Verify that your tunnel is up.

    gcloud compute --project ${PROJECT} vpn-tunnels describe \
        --region ${REGION} ${TUNNEL}

