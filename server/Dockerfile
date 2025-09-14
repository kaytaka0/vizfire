FROM nginx:alpine

RUN apk add --no-cache curl bind-tools iproute2

CMD sh -c "ip route add ${CLIENT_IP} via ${FW_IP} && nginx -g 'daemon off;'"
