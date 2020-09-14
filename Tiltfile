# Load the extension for live updating
load('ext://restart_process', 'docker_build_with_restart')

local_resource(
    'hegel-build',
    'CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o build/hegel ./',
    deps=[
        'go.mod',
        'go.sum',
        'main.go',
        'http_server.go',
        'http_handlers.go',
        'grpc_server.go',
        'xff',
        'metrics',
        'grpc',
        'hegelc'
    ]
)

docker_build_with_restart(
    'quay.io/tinkerbell/hegel',
    '.',
    dockerfile_contents="""
FROM gcr.io/distroless/base:debug as debug
WORKDIR /
COPY build/hegel /hegel
ENTRYPOINT ["/hegel"]
""",
    only=[
        './build/hegel',
    ],
    target='debug',
    live_update=[
        sync('./build/hegel', '/hegel')
    ],
    entrypoint=[
        '/hegel',
    ]
)

k8s_yaml('manifests/hegel.yaml')
k8s_resource(
    workload='hegel',
    resource_deps=[
        'tink-server',
        'tink-mirror',
    ]
)
