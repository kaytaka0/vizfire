FROM alpine:latest

RUN apk add --no-cache curl bind-tools iproute2

CMD sh -c "ip route add ${SERVER_IP} via ${FW_IP} && tail -f /dev/null"
